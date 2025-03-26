---
layout: post
title:  ModelMapper使用指南
category: libraries
copyright: libraries
excerpt: ModelMapper
---

## 1. 概述

在之前的教程中，我们已经了解了如何[使用ModelMapper映射列表](https://www.baeldung.com/java-modelmapper-lists)。

在本教程中，我们将展示**如何在ModelMapper中不同结构的对象之间映射数据**。

虽然ModelMapper的默认转换在典型情况下效果很好，但我们将主要关注如何匹配不够相似的对象而无法使用默认配置处理。

因此，这次我们将目光投向属性映射和配置更改。

## 2. Maven依赖

要开始使用[ModelMapper](https://mvnrepository.com/artifact/org.modelmapper/modelmapper)库，我们将依赖添加到pom.xml中：

```xml

<dependency>
    <groupId>org.modelmapper</groupId>
    <artifactId>modelmapper</artifactId>
    <version>3.2.0</version>
</dependency>
```

## 3. 默认配置

当我们的源对象和目标对象彼此相似时，ModelMapper提供了一个嵌入式解决方案。

让我们分别看一下Game和GameDTO，我们的域对象和相应的数据传输对象：

```java
public class Game {

    private Long id;
    private String name;
    private Long timestamp;

    private Player creator;
    private List<Player> players = new ArrayList<>();

    private GameSettings settings;

    // constructors, getters and setters
}

public class GameDTO {

    private Long id;
    private String name;

    // constructors, getters and setters
}
```

GameDTO仅包含两个字段，但字段类型和名称与源完全匹配。

在这种情况下，ModelMapper无需额外配置即可处理转换：

```java
@BeforeEach
public void setup() {
    this.mapper = new ModelMapper();
}

@Test
public void whenMapGameWithExactMatch_thenConvertsToDTO() {
    // when similar source object is provided
    Game game = new Game(1L, "Game 1");
    GameDTO gameDTO = this.mapper.map(game, GameDTO.class);
    
    // then it maps by default
    assertEquals(game.getId(), gameDTO.getId());
    assertEquals(game.getName(), gameDTO.getName());
}
```

## 4. ModelMapper中的属性映射是什么？

在我们的项目中，大多数时候，我们需要自定义DTO。当然，这会导致不同的字段、层次结构及其相互之间的不规则映射。有时，我们还需要为单个源设置多个DTO，反之亦然。

因此，**属性映射给我们提供了一种扩展映射逻辑的强大方法**。

让我们通过添加新字段creationTime来定制我们的GameDTO：

```java
public class GameDTO {

    private Long id;
    private String name;
    private Long creationTime;

    // constructors, getters and setters
}
```

我们将Game的timestamp字段映射到GameDTO的creationTime字段，注意，**这次源字段名称与目标字段名称不同**。

为了定义属性映射，我们将使用ModelMapper的TypeMap。

因此，让我们创建一个TypeMap对象并通过其addMapping方法添加属性映射：

```java
@Test
public void whenMapGameWithBasicPropertyMapping_thenConvertsToDTO() {
    // setup
    TypeMap<Game, GameDTO> propertyMapper = this.mapper.createTypeMap(Game.class, GameDTO.class);
    propertyMapper.addMapping(Game::getTimestamp, GameDTO::setCreationTime);

    // when field names are different
    Game game = new Game(1L, "Game 1");
    game.setTimestamp(Instant.now().getEpochSecond());
    GameDTO gameDTO = this.mapper.map(game, GameDTO.class);

    // then it maps via property mapper
    assertEquals(game.getId(), gameDTO.getId());
    assertEquals(game.getName(), gameDTO.getName());
    assertEquals(game.getTimestamp(), gameDTO.getCreationTime());
}
```

### 4.1 深度映射

还有不同的映射方式。例如，**ModelMapper可以映射层次结构-不同级别的字段可以进行深度映射**。

让我们在GameDTO中定义一个名为creator的字符串字段。

但是，Game域中的源creator字段不是简单类型，而是一个对象Player：

```java
public class Player {

    private Long id;
    private String name;

    // constructors, getters and setters
}

public class Game {
    // ...

    private Player creator;

    // ...
}

public class GameDTO {
    // ...

    private String creator;

    // ...
}
```

因此，我们不会将整个Player对象的数据传输到GameDTO，而只将name字段传输到GameDTO。

**为了定义深度映射，我们使用TypeMap的addMappings方法并添加一个ExpressionMap**：

```java
@Test
public void whenMapGameWithDeepMapping_thenConvertsToDTO() {
    // setup
    TypeMap<Game, GameDTO> propertyMapper = this.mapper.createTypeMap(Game.class, GameDTO.class);
    // add deep mapping to flatten source's Player object into a single field in destination
    propertyMapper.addMappings(
      mapper -> mapper.map(src -> src.getCreator().getName(), GameDTO::setCreator)
    );
    
    // when map between different hierarchies
    Game game = new Game(1L, "Game 1");
    game.setCreator(new Player(1L, "John"));
    GameDTO gameDTO = this.mapper.map(game, GameDTO.class);
    
    // then
    assertEquals(game.getCreator().getName(), gameDTO.getCreator());
}
```

### 4.2 跳过属性

有时，我们不想暴露DTO中的所有数据，无论是为了让DTO更轻量还是隐藏一些敏感数据，这些原因都可能导致我们在传输到DTO时排除某些字段。

幸运的是，**ModelMapper支持通过跳过来排除属性**。

让我们借助skip方法将id字段排除在传输之外：

```java
@Test
public void whenMapGameWithSkipIdProperty_thenConvertsToDTO() {
    // setup
    TypeMap<Game, GameDTO> propertyMapper = this.mapper.createTypeMap(Game.class, GameDTO.class);
    propertyMapper.addMappings(mapper -> mapper.skip(GameDTO::setId));

    // when id is skipped
    Game game = new Game(1L, "Game 1");
    GameDTO gameDTO = this.mapper.map(game, GameDTO.class);

    // then destination id is null
    assertNull(gameDTO.getId());
    assertEquals(game.getName(), gameDTO.getName());
}
```

因此，GameDTO的id字段被跳过并且不被设置。

### 4.3 Converter

ModelMapper的另一个功能是Converter，**我们可以自定义特定源到目标映射的转换**。

假设我们在Game域中有一个Player集合，让我们将Player的数量转移到GameDTO。

第一步，我们在GameDTO中定义一个整数字段totalPlayers：

```java
public class GameDTO {
    // ...

    private int totalPlayers;

    // constructors, getters and setters
}
```

分别地，我们创建collectionToSize转换器：

```java
Converter<Collection, Integer> collectionToSize = c -> c.getSource().size();
```

最后，我们在添加ExpressionMap时通过使用方法注册我们的Converter：

```java
propertyMapper.addMappings(
    mapper -> mapper.using(collectionToSize).map(Game::getPlayers, GameDTO::setTotalPlayers)
);
```

因此，我们将Game的getPlayers().size()映射到GameDTO的totalPlayers字段：

```java
@Test
public void whenMapGameWithCustomConverter_thenConvertsToDTO() {
    // setup
    TypeMap<Game, GameDTO> propertyMapper = this.mapper.createTypeMap(Game.class, GameDTO.class);
    Converter<Collection, Integer> collectionToSize = c -> c.getSource().size();
    propertyMapper.addMappings(
            mapper -> mapper.using(collectionToSize).map(Game::getPlayers, GameDTO::setTotalPlayers)
    );

    // when collection to size converter is provided
    Game game = new Game();
    game.addPlayer(new Player(1L, "John"));
    game.addPlayer(new Player(2L, "Bob"));
    GameDTO gameDTO = this.mapper.map(game, GameDTO.class);

    // then it maps the size to a custom field
    assertEquals(2, gameDTO.getTotalPlayers());
}
```

### 4.4 Provider

在另一个用例中，我们有时需要为目标对象提供一个实例，而不是让ModalMapper初始化它，这时Provider就派上用场了。

因此，**ModelMapper的Provider是自定义目标对象实例化的内置方法**。

让我们进行一次转换，这次不是Game到DTO，而是Game到Game。

所以，原则上，我们有一个持久化的Game域，我们从它的存储库中获取它。

之后，我们通过将另一个Game对象合并到Game实例中来更新该Game实例：

```java
@Test
public void whenUsingProvider_thenMergesGameInstances() {
    // setup
    TypeMap<Game, Game> propertyMapper = this.mapper.createTypeMap(Game.class, Game.class);
    // a provider to fetch a Game instance from a repository
    Provider<Game> gameProvider = p -> this.gameRepository.findById(1L);
    propertyMapper.setProvider(gameProvider);
    
    // when a state for update is given
    Game update = new Game(1L, "Game Updated!");
    update.setCreator(new Player(1L, "John"));
    Game updatedGame = this.mapper.map(update, Game.class);
    
    // then it merges the updates over on the provided instance
    assertEquals(1L, updatedGame.getId().longValue());
    assertEquals("Game Updated!", updatedGame.getName());
    assertEquals("John", updatedGame.getCreator().getName());
}
```

### 4.5 条件映射

**ModelMapper还支持条件映射**，我们可以使用的内置条件方法之一是Conditions.isNull()。

如果源Game对象中的id字段为空，我们就跳过它：

```java
@Test
public void whenUsingConditionalIsNull_thenMergesGameInstancesWithoutOverridingId() {
    // setup
    TypeMap<Game, Game> propertyMapper = this.mapper.createTypeMap(Game.class, Game.class);
    propertyMapper.setProvider(p -> this.gameRepository.findById(2L));
    propertyMapper.addMappings(mapper -> mapper.when(Conditions.isNull()).skip(Game::getId, Game::setId));
    
    // when game has no id
    Game update = new Game(null, "Not Persisted Game!");
    Game updatedGame = this.mapper.map(update, Game.class);
    
    // then destination game id is not overwritten
    assertEquals(2L, updatedGame.getId().longValue());
    assertEquals("Not Persisted Game!", updatedGame.getName());
}
```

请注意，通过使用isNull条件与skip方法结合，我们可以防止目标ID被空值覆盖。

此外，**我们还可以定义自定义的Condition**。

让我们定义一个条件来检查Game的timestamp字段是否有值：

```java
Condition<Long, Long> hasTimestamp = ctx -> ctx.getSource() != null && ctx.getSource() > 0;
```

接下来，我们在属性映射器中使用when方法：

```java
TypeMap<Game, GameDTO> propertyMapper = this.mapper.createTypeMap(Game.class, GameDTO.class);
Condition<Long, Long> hasTimestamp = ctx -> ctx.getSource() != null && ctx.getSource() > 0;
propertyMapper.addMappings(
    mapper -> mapper.when(hasTimestamp).map(Game::getTimestamp, GameDTO::setCreationTime)
);
```

最后，如果timestamp的值大于0，ModelMapper仅更新GameDTO的creationTime字段：

```java
@Test
public void whenUsingCustomConditional_thenConvertsDTOSkipsZeroTimestamp() {
    // setup
    TypeMap<Game, GameDTO> propertyMapper = this.mapper.createTypeMap(Game.class, GameDTO.class);
    Condition<Long, Long> hasTimestamp = ctx -> ctx.getSource() != null && ctx.getSource() > 0;
    propertyMapper.addMappings(
        mapper -> mapper.when(hasTimestamp).map(Game::getTimestamp, GameDTO::setCreationTime)
    );
    
    // when game has zero timestamp
    Game game = new Game(1L, "Game 1");
    game.setTimestamp(0L);
    GameDTO gameDTO = this.mapper.map(game, GameDTO.class);
    
    // then timestamp field is not mapped
    assertEquals(game.getId(), gameDTO.getId());
    assertEquals(game.getName(), gameDTO.getName());
    assertNotEquals(0L ,gameDTO.getCreationTime());
    
    // when game has timestamp greater than zero
    game.setTimestamp(Instant.now().getEpochSecond());
    gameDTO = this.mapper.map(game, GameDTO.class);
    
    // then timestamp field is mapped
    assertEquals(game.getId(), gameDTO.getId());
    assertEquals(game.getName(), gameDTO.getName());
    assertEquals(game.getTimestamp() ,gameDTO.getCreationTime());
}
```

## 5. 其他映射方法

在大多数情况下，属性映射是一种很好的方法，因为它允许我们做出明确的定义并清楚地看到映射的流程。

然而，对于某些对象，特别是当它们具有不同的属性层次结构时，**我们可以使用LOOSE匹配策略来代替TypeMap**。

### 5.1 匹配策略LOOSE

为了展示松散匹配的好处，我们在GameDTO中添加另外两个属性：

```java
public class GameDTO {
    //...
    
    private GameMode mode;
    private int maxPlayers;
    
    // constructors, getters and setters
}
```

请注意，mode和maxPlayers对应于GameSettings的属性，它是我们的Game源类中的内部对象：

```java
public class GameSettings {

    private GameMode mode;
    private int maxPlayers;

    // constructors, getters and setters
}
```

这样，我们就可以执行双向映射，从Game到GameDTO以及反之亦然，而无需定义任何TypeMap：

```java
@Test
public void whenUsingLooseMappingStrategy_thenConvertsToDomainAndDTO() {
    // setup
    this.mapper.getConfiguration().setMatchingStrategy(MatchingStrategies.LOOSE);
    
    // when dto has flat fields for GameSetting
    GameDTO gameDTO = new GameDTO();
    gameDTO.setMode(GameMode.TURBO);
    gameDTO.setMaxPlayers(8);
    Game game = this.mapper.map(gameDTO, Game.class);
    
    // then it converts to inner objects without property mapper
    assertEquals(gameDTO.getMode(), game.getSettings().getMode());
    assertEquals(gameDTO.getMaxPlayers(), game.getSettings().getMaxPlayers());
    
    // when the GameSetting's field names match
    game = new Game();
    game.setSettings(new GameSettings(GameMode.NORMAL, 6));
    gameDTO = this.mapper.map(game, GameDTO.class);
    
    // then it flattens the fields on dto
    assertEquals(game.getSettings().getMode(), gameDTO.getMode());
    assertEquals(game.getSettings().getMaxPlayers(), gameDTO.getMaxPlayers());
}
```

### 5.2 自动跳过空属性

此外，ModelMapper还有一些有用的全局配置，其中之一就是setSkipNullEnabled设置。

因此，**如果源属性为空，我们可以自动跳过它们，而无需编写任何[条件映射](https://www.baeldung.com/java-modelmapper#5-conditional-mapping)**：

```java
@Test
public void whenConfigurationSkipNullEnabled_thenConvertsToDTO() {
    // setup
    this.mapper.getConfiguration().setSkipNullEnabled(true);
    TypeMap<Game, Game> propertyMap = this.mapper.createTypeMap(Game.class, Game.class);
    propertyMap.setProvider(p -> this.gameRepository.findById(2L));
    
    // when game has no id
    Game update = new Game(null, "Not Persisted Game!");
    Game updatedGame = this.mapper.map(update, Game.class);
    
    // then destination game id is not overwritten
    assertEquals(2L, updatedGame.getId().longValue());
    assertEquals("Not Persisted Game!", updatedGame.getName());
}
```

### 5.3 循环引用对象

有时，我们需要处理具有对自身的引用的对象。

通常，这会导致循环依赖并引发著名的StackOverflowError：

```text
org.modelmapper.MappingException: ModelMapper mapping errors:

1) Error mapping cn.tuyucheng.taketoday.domain.Game to cn.tuyucheng.taketoday.dto.GameDTO

1 error
	...
Caused by: java.lang.StackOverflowError
	...
```

因此，另一个配置setPreferNestedProperties将在这种情况下帮助我们：

```java
@Test
public void whenConfigurationPreferNestedPropertiesDisabled_thenConvertsCircularReferencedToDTO() {
    // setup
    this.mapper.getConfiguration().setPreferNestedProperties(false);
    
    // when game has circular reference: Game -> Player -> Game
    Game game = new Game(1L, "Game 1");
    Player player = new Player(1L, "John");
    player.setCurrentGame(game);
    game.setCreator(player);
    GameDTO gameDTO = this.mapper.map(game, GameDTO.class);
    
    // then it resolves without any exception
    assertEquals(game.getId(), gameDTO.getId());
    assertEquals(game.getName(), gameDTO.getName());
}
```

因此，当我们将false传递给setPreferNestedProperties时，映射将有效并且不会出现任何异常。

## 6. 总结

在本文中，我们解释了如何使用ModelMapper中的属性映射器自定义类到类的映射。

我们还看到了一些替代配置的详细示例。