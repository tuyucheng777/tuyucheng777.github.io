---
layout: post
title:  在Spring应用程序中通过CSV外部化设置数据
category: springdata
copyright: springdata
excerpt: Spring Data JPA
---

## 1. 概述

在本文中，我们将**使用CSV文件外部化应用程序的设置数据**，而不是对其进行硬编码。

此设置过程主要涉及在新的系统上设置新数据。

## 2. CSV库

让我们首先介绍一个用于处理CSV的简单库-[Jackson CSV扩展](https://github.com/FasterXML/jackson)：

```xml
<dependency>
    <groupId>com.fasterxml.jackson.dataformat</groupId>
    <artifactId>jackson-dataformat-csv</artifactId>       
    <version>2.5.3</version>
</dependency>
```

当然，Java生态系统中有大量可用于处理CSV的库。

我们在这里选择Jackson的原因是-Jackson很可能已经在应用程序中使用，并且我们读取数据所需的处理相当简单。

## 3. 设置数据

不同的项目需要设置不同的数据。

在本教程中，我们将设置用户数据-基本上用几个默认用户准备系统。

以下是包含用户的简单CSV文件：

```csv
id,username,password,accessToken
1,john,123,token
2,tom,456,test
```

请注意文件的第一行是标题行-列出每行数据中字段的名称。

## 3. CSV数据加载器

让我们首先创建一个简单的数据加载器，**将CSV文件中的数据读入工作内存**。

### 3.1 加载对象列表

我们将实现loadObjectList()功能来从文件中加载完全参数化的特定对象列表：

```java
public <T> List<T> loadObjectList(Class<T> type, String fileName) {
    try {
        CsvSchema bootstrapSchema = CsvSchema.emptySchema().withHeader();
        CsvMapper mapper = new CsvMapper();
        File file = new ClassPathResource(fileName).getFile();
        MappingIterator<T> readValues = mapper.reader(type).with(bootstrapSchema).readValues(file);
        return readValues.readAll();
    } catch (Exception e) {
        logger.error("Error occurred while loading object list from file " + fileName, e);
        return Collections.emptyList();
    }
}
```

注意：

- 我们根据第一个“标题”行创建了CSVSchema
- 该实现足够通用，可以处理任何类型的对象
- 如果发生任何错误，将返回一个空列表

### 3.2 处理多对多关系

Jackson CSV不太支持嵌套对象-我们需要使用间接的方式来加载多对多关系。

我们将这些表示为**类似于简单的连接表**-因此自然地我们会将其从磁盘加载为数组列表：

```java
public List<String[]> loadManyToManyRelationship(String fileName) {
    try {
        CsvMapper mapper = new CsvMapper();
        CsvSchema bootstrapSchema = CsvSchema.emptySchema().withSkipFirstDataRow(true);
        mapper.enable(CsvParser.Feature.WRAP_AS_ARRAY);
        File file = new ClassPathResource(fileName).getFile();
        MappingIterator<String[]> readValues =
                mapper.reader(String[].class).with(bootstrapSchema).readValues(file);
        return readValues.readAll();
    } catch (Exception e) {
        logger.error(
                "Error occurred while loading many to many relationship from file = " + fileName, e);
        return Collections.emptyList();
    }
}
```

以下是这些关系之一(角色<->权限)在简单的CSV文件中的表示方式：

```csv
role,privilege
ROLE_ADMIN,ADMIN_READ_PRIVILEGE
ROLE_ADMIN,ADMIN_WRITE_PRIVILEGE
ROLE_SUPER_USER,POST_UNLIMITED_PRIVILEGE
ROLE_USER,POST_LIMITED_PRIVILEGE
```

请注意我们在这个实现中是如何忽略标题的，因为我们实际上不需要这些信息。

## 4. 设置数据

现在，我们将使用一个简单的Setup Bean来完成从CSV文件设置权限、角色和用户的所有工作：

```java
@Component
public class Setup {
    // ...

    @PostConstruct
    private void setupData() {
        setupRolesAndPrivileges();
        setupUsers();
    }

    // ...
}
```

### 4.1 设置角色和权限

首先，让我们将角色和权限从磁盘加载到工作内存中，然后将它们作为设置过程的一部分保存下来：

```java
public List<Privilege> getPrivileges() {
    return csvDataLoader.loadObjectList(Privilege.class, PRIVILEGES_FILE);
}

public List<Role> getRoles() {
    List<Privilege> allPrivileges = getPrivileges();
    List<Role> roles = csvDataLoader.loadObjectList(Role.class, ROLES_FILE);
    List<String[]> rolesPrivileges = csvDataLoader.
            loadManyToManyRelationship(SetupData.ROLES_PRIVILEGES_FILE);

    for (String[] rolePrivilege : rolesPrivileges) {
        Role role = findRoleByName(roles, rolePrivilege[0]);
        Set<Privilege> privileges = role.getPrivileges();
        if (privileges == null) {
            privileges = new HashSet<Privilege>();
        }
        privileges.add(findPrivilegeByName(allPrivileges, rolePrivilege[1]));
        role.setPrivileges(privileges);
    }
    return roles;
}

private Role findRoleByName(List<Role> roles, String roleName) {
    return roles.stream().
            filter(item -> item.getName().equals(roleName)).findFirst().get();
}

private Privilege findPrivilegeByName(List<Privilege> allPrivileges, String privilegeName) {
    return allPrivileges.stream().
            filter(item -> item.getName().equals(privilegeName)).findFirst().get();
}
```

然后我们在这里进行持久化工作：

```java
private void setupRolesAndPrivileges() {
    List<Privilege> privileges = setupData.getPrivileges();
    for (Privilege privilege : privileges) {
        setupService.setupPrivilege(privilege);
    }

    List<Role> roles = setupData.getRoles();
    for (Role role : roles) {
        setupService.setupRole(role);
    }
}
```

这是我们的SetupService：

```java
public void setupPrivilege(Privilege privilege) {
    if (privilegeRepository.findByName(privilege.getName()) == null) {
        privilegeRepository.save(privilege);
    }
}

public void setupRole(Role role) {
    if (roleRepository.findByName(role.getName()) == null) {
        Set<Privilege> privileges = role.getPrivileges();
        Set<Privilege> persistedPrivileges = new HashSet<Privilege>();
        for (Privilege privilege : privileges) {
            persistedPrivileges.add(privilegeRepository.findByName(privilege.getName()));
        }
        role.setPrivileges(persistedPrivileges);
        roleRepository.save(role); }
}
```

请注意，我们将角色和权限加载到工作内存后，会逐一加载它们的关系。

### 4.2 设置初始用户

接下来，让我们将用户加载到内存中并保存起来：

```java
public List<User> getUsers() {
    List<Role> allRoles = getRoles();
    List<User> users = csvDataLoader.loadObjectList(User.class, SetupData.USERS_FILE);
    List<String[]> usersRoles = csvDataLoader.
            loadManyToManyRelationship(SetupData.USERS_ROLES_FILE);

    for (String[] userRole : usersRoles) {
        User user = findByUserByUsername(users, userRole[0]);
        Set<Role> roles = user.getRoles();
        if (roles == null) {
            roles = new HashSet<Role>();
        }
        roles.add(findRoleByName(allRoles, userRole[1]));
        user.setRoles(roles);
    }
    return users;
}

private User findByUserByUsername(List<User> users, String username) {
    return users.stream().
            filter(item -> item.getUsername().equals(username)).findFirst().get();
}
```

接下来，让我们重点关注持久化用户：

```java
private void setupUsers() {
    List<User> users = setupData.getUsers();
    for (User user : users) {
        setupService.setupUser(user);
    }
}
```

这是我们的SetupService：

```java
@Transactional
public void setupUser(User user) {
    try {
        setupUserInternal(user);
    } catch (Exception e) {
        logger.error("Error occurred while saving user " + user.toString(), e);
    }
}

private void setupUserInternal(User user) {
    if (userRepository.findByUsername(user.getUsername()) == null) {
        user.setPassword(passwordEncoder.encode(user.getPassword()));
        user.setPreference(createSimplePreference(user));
        Set<Role> roles = user.getRoles();
        Set<Role> persistedRoles = new HashSet<Role>();
        for (Role role : roles) {
            persistedRoles.add(roleRepository.findByName(role.getName()));
        }
        user.setRoles(persistedRoles);
        userRepository.save(user);
    }
}
```

以下是createSimplePreference()方法：

```java
private Preference createSimplePreference(User user) {
    Preference pref = new Preference();
    pref.setId(user.getId());
    pref.setTimezone(TimeZone.getDefault().getID());
    pref.setEmail(user.getUsername() + "@test.com");
    return preferenceRepository.save(pref);
}
```

请注意，在保存用户之前，我们先为其创建一个简单的Preference实体并首先将其持久化。

## 5. 测试CSV数据加载器

接下来，让我们对CsvDataLoader执行一个简单的单元测试：

我们将测试加载用户、角色和权限列表：

```java
@Test
public void whenLoadingUsersFromCsvFile_thenLoaded() {
    List<User> users = csvDataLoader.loadObjectList(User.class, CsvDataLoader.USERS_FILE);
    assertFalse(users.isEmpty());
}

@Test
public void whenLoadingRolesFromCsvFile_thenLoaded() {
    List<Role> roles = csvDataLoader.loadObjectList(Role.class, CsvDataLoader.ROLES_FILE);
    assertFalse(roles.isEmpty());
}

@Test
public void whenLoadingPrivilegesFromCsvFile_thenLoaded() {
    List<Privilege> privileges = csvDataLoader.loadObjectList(Privilege.class, CsvDataLoader.PRIVILEGES_FILE);
    assertFalse(privileges.isEmpty());
}
```

接下来，让我们测试通过数据加载器加载一些多对多关系：

```java
@Test
public void whenLoadingUsersRolesRelationFromCsvFile_thenLoaded() {
    List<String[]> usersRoles = csvDataLoader.loadManyToManyRelationship(CsvDataLoader.USERS_ROLES_FILE);
    assertFalse(usersRoles.isEmpty());
}

@Test
public void whenLoadingRolesPrivilegesRelationFromCsvFile_thenLoaded() {
    List<String[]> rolesPrivileges = csvDataLoader.loadManyToManyRelationship(CsvDataLoader.ROLES_PRIVILEGES_FILE);
    assertFalse(rolesPrivileges.isEmpty());
}
```

## 6. 测试设置数据

最后，让我们对Bean SetupData执行一个简单的单元测试：

```java
@Test
public void whenGettingUsersFromCsvFile_thenCorrect() {
    List<User> users = setupData.getUsers();

    assertFalse(users.isEmpty());
    for (User user : users) {
        assertFalse(user.getRoles().isEmpty());
    }
}

@Test
public void whenGettingRolesFromCsvFile_thenCorrect() {
    List<Role> roles = setupData.getRoles();

    assertFalse(roles.isEmpty());
    for (Role role : roles) {
        assertFalse(role.getPrivileges().isEmpty());
    }
}

@Test
public void whenGettingPrivilegesFromCsvFile_thenCorrect() {
    List<Privilege> privileges = setupData.getPrivileges();
    assertFalse(privileges.isEmpty());
}
```

## 7. 总结

在这篇简短的文章中，我们探讨了一种用于初始数据的替代设置方法，这些数据通常需要在启动时加载到系统中。这当然只是一个简单的概念验证和一个良好的基础，而不是一个可用于生产的解决方案。