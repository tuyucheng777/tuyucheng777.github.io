---
layout: post
title:  在Spring Boot项目中禁用Hibernate实体验证
category: springboot
copyright: springboot
excerpt: Spring Boot
---

## 1. 概述

在使用Hibernate的Spring Boot项目中，[实体验证](https://www.baeldung.com/spring-boot-bean-validation)通常在持久化期间自动应用。

虽然Hibernate的内置验证很有用，但如果我们的控制器已经处理了所有必要的验证检查，它就会变得多余。**这种双重验证设置会导致重复验证**，从而产生不必要的开销。此外，**当使用依赖于Spring Bean的自定义验证器时，它可能会导致依赖注入问题**。

通过专门针对JPA实体禁用Hibernate验证，我们可以避免冗余检查，并将验证逻辑集中在控制器层(如果我们的控制器已经有效地处理了验证逻辑)。由于验证变得集中，因此这可以提高性能并简化代码库。

本教程介绍禁用Hibernate的JPA实体验证的步骤，强调此方法的好处及其对应用程序效率的积极影响。

## 2. 为什么要禁用Hibernate验证？

让我们探讨在Spring Boot应用程序中禁用Hibernate验证的主要原因，虽然Hibernate的自动验证很方便，但如果我们的控制器已经有效地处理验证，那么它可能就没有必要了。

- **[避免冗余检查](https://www.baeldung.com/spring-valid-vs-validated)**：当我们在控制器中完全处理验证时，Hibernate的实体级验证会重复此过程，从而导致资源浪费。每次我们持久化一个实体时，Hibernate都会执行控制器已经处理过的验证检查，从而导致不必要的性能成本。
- **预防依赖注入问题**：Hibernate验证可能会导致依赖注入问题，尤其是使用自定义验证器时。例如，如果自定义验证器注入了Spring组件(例如用于检查唯一性约束的Repository)，Hibernate验证将无法识别这些注入的依赖项。禁用Hibernate验证可使这些验证器在Spring框架内顺利运行。
- **提高性能**：禁用冗余的Hibernate检查可提高应用程序性能，仅依靠基于控制器的验证可减少持久性期间的处理负载，这对于处理高交易量或大型数据集的应用程序至关重要。

简而言之，如果我们已经在控制器内全面管理验证，那么禁用Hibernate的验证可以使我们的应用程序更精简、更高效。

## 3. 在控制器层专门设置验证

通过[在控制器层配置验证](https://www.baeldung.com/spring-validate-requestparam-pathvariable)，我们**确保数据在到达持久层之前得到正确验证**。这种方法使我们的验证保持集中化，并更好地控制进入我们应用程序的数据流。

让我们在控制器中实现验证：

```java
@PostMapping
public ResponseEntity<String> addAppUser(@Valid @RequestBody AppUser appUser, BindingResult result) {
    if (result.hasErrors()) {
        return new ResponseEntity<>(result.getFieldError().getDefaultMessage(), HttpStatus.BAD_REQUEST);
    }
    appUserRepository.save(appUser);
    return new ResponseEntity<>("AppUser created successfully", HttpStatus.CREATED);
}
```

在此示例中，我们使用@Valid注解标注BindingResult参数直接在控制器中捕获验证错误。让我们分解一下此设置：

- @Valid注解：根据AppUser实体上定义的约束触发验证(例如@NotNull、@Size或自定义注解)，此设置可确保验证发生在控制器级别，而不是在持久化期间。
- BindingResult：此参数捕获任何验证错误，使我们能够在实体持久化之前管理这些错误。这样，如果AppUser实体验证失败，BindingResult将包含错误详细信息，我们可以相应地处理它们。

通过在控制器中设置验证，我们将所有数据检查集中在应用程序的入口点，从而降低了无效数据到达数据库的风险。

## 4. 在application.properties中禁用Hibernate实体验证

要禁用JPA实体的Hibernate验证，我们需要修改Spring Boot项目中的[application.properties](https://www.baeldung.com/properties-with-spring)文件。通过设置单个属性，我们可以完全禁用Hibernate的实体验证机制，仅依赖于基于控制器的验证：

```properties
spring.jpa.properties.jakarta.persistence.validation.mode=none
```

此设置指示Hibernate在持久化期间跳过实体验证，jakarta.persistence.validation.mode属性提供了三个选项：

- **auto**：如果Hibernate检测到验证配置，则验证自动运行。
- **callback**：仅通过回调明确触发时才会发生验证。
- **none**：此选项完全禁用Hibernate实体验证，将验证留给其他应用程序层(在本例中为我们的控制器层)。

将此属性设置为none会指示Hibernate忽略实体上的任何验证注解，从而允许我们在控制器中专门管理验证。

## 5. 自定义验证器和依赖注入

当我们禁用Hibernate验证时，我们可以放心地使用依赖于Spring依赖注入的自定义验证器。自定义验证器提供了一种强大的方法来强制执行特定的业务规则或验证逻辑，特别是对于唯一约束。

让我们看一个自定义验证器的例子：

```java
public class UserUniqueValidator implements ConstraintValidator<UserUnique, String> {
    @Autowired
    private AppUserRepository appUserRepository;

    @Override
    public boolean isValid(String username, ConstraintValidatorContext context) {
        return appUserRepository.findByUsername(username) == null;
    }
}
```

此示例显示了一个自定义验证器，它通过查询Repository来检查用户名是否唯一。如果Hibernate验证处于激活状态，则此验证器可能会因依赖项注入问题而失败(例如，AppUserRepository可能未正确注入)。

但是，**禁用Hibernate验证后，该验证器将在控制器内独立运行，并使用Spring的依赖注入来确保AppUserRepository可用**。

## 6. 了解实体和模式验证之间的区别

区分实体验证(检查数据约束，如@NotNull或@Size)和模式验证(将实体结构与数据库模式同步)至关重要，禁用Hibernate的验证模式只会影响实体级验证，而不会影响模式验证。

Hibernate中的模式验证会检查数据库模式是否与应用程序中定义的实体映射相匹配。例如，如果我们更改实体类(例如重命名字段或更改列长度)，模式验证会检测实体定义和数据库模式之间的差异。

## 7. 在application.properties中禁用模式验证

如果我们希望Hibernate也跳过模式验证(意味着它不会自动验证或修改数据库模式)，我们可以在application.properties文件中调整相应的属性：

```properties
spring.jpa.hibernate.ddl-auto=none
```

使用此设置，Hibernate不再执行模式验证或数据库创建/更改操作。如果我们使用[Flyway](https://www.baeldung.com/database-migrations-with-flyway)或[Liquibase](https://www.baeldung.com/liquibase-refactor-schema-of-java-app)等专用迁移工具独立管理数据库模式，此设置特别有用。

## 8. 禁用Hibernate验证后测试设置

配置Hibernate忽略实体验证后，必须验证校验在控制器层中是否正常运行，此测试步骤可确保我们的验证在没有Hibernate参与的情况下仍然有效。

让我们回顾一下测试设置所需采取的步骤：

- **功能测试**：向控制器端点发送各种有效和无效的数据输入，以确认正确捕获了验证错误，此步骤可验证控制器级校验设置是否按预期运行。
- **日志验证**：检查应用程序日志以确保不存在Hibernate验证错误，现在所有验证错误消息都应来自控制器，这表明我们已成功禁用Hibernate验证。

此测试阶段确认我们的应用程序仅通过控制器层处理验证，并且Hibernate在持久化期间不会重新引入验证检查。

## 9. 全局管理验证异常

使用@ControllerAdvice提供了一种在Spring Boot中全局处理验证错误的有效方法，确保整个应用程序的响应一致。

让我们看一个全局异常处理程序的示例：

```java
@ControllerAdvice
public class GlobalExceptionHandler {
    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<String> handleValidationException(MethodArgumentNotValidException ex) {
        return new ResponseEntity<>("Validation error: " + ex.getMessage(), HttpStatus.BAD_REQUEST);
    }
}
```

通过此设置，我们可以更有效地管理验证错误，为无效数据输入提供清晰一致的错误响应。此方法通过清晰地传达验证错误来增强用户体验。

## 10. 禁用实体验证的主要好处

通过禁用实体验证，我们可以获得多个优势，从而提高应用程序的性能、可维护性和整体效率：

- **集中验证逻辑**：通过在控制器层专门处理验证，所有验证规则都在一个地方管理，从而简化了代码库并提高了可维护性。
- **减少冗余**：删除重复检查可消除验证结果不一致的风险并防止不必要的处理。
- **增强的性能**：更少的验证检查可缩短处理时间，这对于性能至关重要的高流量应用程序尤其有益。

## 11. 总结

在Spring Boot中禁用实体验证是一种实用的优化，可简化验证管理、提高应用程序性能并最大限度地降低复杂性。通过将验证逻辑集中在控制器层，我们可以保持强大的数据完整性，同时避免与依赖注入和冗余检查相关的潜在陷阱。

这种方法为具有高性能需求或复杂验证规则的应用程序提供了更精简、更高效的解决方案，使我们能够更好地控制验证而不影响功能。