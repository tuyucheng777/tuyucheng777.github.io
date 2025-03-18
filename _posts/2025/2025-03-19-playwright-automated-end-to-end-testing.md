---
layout: post
title:  使用Playwright进行自动化端到端测试
category: automatedtest
copyright: automatedtest
excerpt: Playwright
---

## 1. 概述

端到端测试是确定软件产品整体运行情况的重要因素之一，它有助于发现单元测试和集成测试阶段可能被忽视的问题，并有助于确定软件是否按预期运行。

执行可能涉及多个用户步骤和旅程的端到端测试非常繁琐。因此，一种可行的方法是执行端到端测试用例的[自动化测试](https://www.lambdatest.com/automation-testing?utm_source=baeldung&utm_medium=bdblog&utm_campaign=gpjuly24)。

在本文中，我们将学习如何使用Playwright和TypeScript自动化端到端测试。

## 2. 什么是Playwright端到端测试？

Playwright端到端测试是帮助开发人员和测试人员Mock用户与网站的真实交互的过程。借助Playwright，我们可以自动执行单击按钮、填写表单和浏览页面等任务，以检查一切是否按预期运行。它适用于Chrome、Firefox、Safari和Edge等热门浏览器。

## 3. Playwright端到端测试的先决条件

要使用Playwright，请**安装NodeJS 18或更高版本和TypeScript**。有两种方法可以安装Playwright：

- 使用命令行
- 使用VS Code

但是，在本文中，让我们使用VS Code来安装 laywright。

1. 从VS Code插件市场安装Playwright后，让我们打开命令面板并运行命令Install Playwright：

![](/assets/images/2025/automatedtest/playwrightautomatedendtoendtesting01.png)

2. 让我们安装所需的浏览器。然后单击“OK”：

![](/assets/images/2025/automatedtest/playwrightautomatedendtoendtesting02.png)

3. 安装完成后我们将在package.json文件中得到包含依赖项的文件夹结构：

![](/assets/images/2025/automatedtest/playwrightautomatedendtoendtesting03.png)

## 4. 如何使用Playwright进行端到端测试？

端到端测试涵盖了最终用户理想情况下遵循的用例，让我们考虑使用[LambdaTest电子商务](https://ecommerce-playground.lambdatest.io/?utm_source=baeldung&utm_medium=bdblog&utm_campaign=gpjuly24)网站来编写端到端测试。

**我们将使用LambdaTest等基于云的测试平台来实现端到端测试的更高可扩展性和可靠性**。[LambdaTest](https://www.lambdatest.com/?utm_source=baeldung&utm_medium=bdblog&utm_campaign=gpjuly24)是一个由人工智能驱动的测试执行平台，使用Playwright在3000多种真实浏览器和操作系统上提供自动化测试。

### 4.1 测试场景1

1. 在LambdaTest电子商务网站上注册新用户
2. 执行断言来检查用户是否已注册成功

### 4.2 测试场景2

1. 执行断言来检查用户是否已登录
2. 在主页上搜索产品
3. 选择产品并将其添加到购物车
4. 执行断言来检查正确的产品是否添加到购物车中

### 4.3 测试配置

让我们创建一个Fixture文件，通过覆盖storageState Fixture来对每个工作器进行一次身份验证。我们可以使用testInfo.parallelIndex来区分工作器。

此外，我们可以使用相同的Fixture文件来配置LambdaTest功能。现在，让我们创建一个名为base的新文件夹和一个新的文件page-object-model-fixture.ts。

第一个块包含来自其他项目目录的npm包和文件的导入语句。我们将导入expect、chromium和test作为baseTest变量，并使用dotenv获取环境变量。然后，我们直接在Fixture文件和test中声明页面对象类实例。

下一个步骤涉及添加LambdaTest功能：

```typescript
const modifyCapabilities = (configName, testName) => {
    let config = configName.split("@lambdatest")[0];
    let [browserName, browserVersion, platform] = config.split(":");
    capabilities.browserName = browserName;
    capabilities.browserVersion = browserVersion;
    capabilities["LT:Options"]["platform"] =
        platform || capabilities["LT:Options"]["platform"];
    capabilities["LT:Options"]["name"] = testName;
};
```

**我们可以使用[LambdaTest功能生成器](https://www.lambdatest.com/capabilities-generator/?utm_source=baeldung&utm_medium=bdblog&utm_campaign=gpjuly24)轻松生成这些功能**。下一行代码将通过自定义和创建项目名称来使用LambdaTest功能。理想情况下，项目名称是浏览器、浏览器版本和平台名称的组合，格式为chrome:latest:macOS Sonoma@lambdatest:

```typescript
projects: [
    {
        name: "chrome:latest:macOS Sonoma@lambdatest",
        use: {
            viewport: {
                width: 1920,
                height: 1080,
            },
        },
    },
    {
        name: "chrome:latest:Windows 10@lambdatest",
        use: {
            viewport: {
                width: 1280,
                height: 720,
            },
        },
    },
```

下一个代码块分为两部分。在第一部分中，声明了testPages常量变量，并将其分配给baseTest，它扩展了fixture文件中最初声明的pages类型以及workerStorageState：

```typescript
const testPages = baseTest.extend<pages, { workerStorageState: string; }>({
    page: async ({}, use, testInfo) => {
        if (testInfo.project.name.match(/lambdatest/)) {
            modifyCapabilities(testInfo.project.name, `${testInfo.title}`);
            const browser =
                await chromium.connect(
                    `wss://cdp.lambdatest.com/playwright?capabilities=
                    ${encodeURIComponent(JSON.stringify(capabilities))}`
                );
            const context = await browser.newContext(testInfo.project.use);
            const ltPage = await context.newPage();
            await use(ltPage);

            const testStatus = {
                action: "setTestStatus",
                arguments: {
                    status: testInfo.status,
                    remark: getErrorMessage(testInfo, ["error", "message"]),
                },
            };
            await ltPage.evaluate(() => {},
                `lambdatest_action: ${JSON.stringify(testStatus)}`
            );
            await ltPage.close();
            await context.close();
            await browser.close();
        } else {
            const browser = await chromium.launch();
            const context = await browser.newContext();
            const page = await context.newPage();
            await use(page);
        }
    },

    homePage: async ({ page }, use) => {
        await use(new HomePage(page));
    },
    registrationPage: async ({ page }, use) => {
        await use(new RegistrationPage(page));
    },
});
```

在块的第二部分中，设置了workerStorageState，每个并行工作器都会进行一次身份验证。所有测试都使用工作器运行时的相同身份验证状态：

```typescript
storageState: ({ workerStorageState }, use) =>
    use(workerStorageState),

workerStorageState: [
    async ({ browser }, use) => {
        const id = test.info().parallelIndex;
        const fileName = path.resolve(
            test.info().project.outputDir,
            `.auth/${id}.json`
        );
    },
],
```

每个工作器都会使用工作器作用域的fixture进行一次身份验证，我们需要通过取消设置存储状态来确保在干净的环境中进行身份验证：

```typescript
const page = await browser.newPage({ storageState: undefined });
```

接下来，应在fixture文件中更新身份验证过程。它包括用户注册步骤，如测试场景1中所述。

### 4.4 实现：测试场景1 

首先，我们将创建两个页面对象类来保存定位器和与每个页面元素交互的函数。让我们在tests文件夹中创建一个名为pageobjects的新文件夹，第一个页面对象类将用于主页：

```typescript
import { Page, Locator } from "@playwright/test";
import { SearchResultPage } from "./search-result-page";

export class HomePage {
    readonly myAccountLink: Locator;
    readonly registerLink: Locator;
    readonly searchProductField: Locator;
    readonly searchBtn: Locator;
    readonly logoutLink: Locator;
    readonly page: Page;

    constructor(page: Page) {
        this.page = page;
        this.myAccountLink = page.getByRole("button", { name: " My account" });
        this.registerLink = page.getByRole("link", { name: "Register" });
        this.logoutLink = page.getByRole("link", { name: " Logout" });
        this.searchProductField = page.getByPlaceholder("Search For Products");
        this.searchBtn = page.getByRole("button", { name: "Search" });
    }

    async hoverMyAccountLink(): Promise<void> {
        await this.myAccountLink.hover({ force: true });
    }

    async navigateToRegistrationPage(): Promise<void> {
        await this.hoverMyAccountLink();
        await this.registerLink.click();
    }
}
```

在主页上，我们首先需要将鼠标悬停在“My account”链接上以打开菜单下拉菜单，然后单击注册链接以打开注册页面：

![](/assets/images/2025/automatedtest/playwrightautomatedendtoendtesting04.png)

在Chrome “DevTools”窗口中，“My account” WebElement角色是一个按钮。因此，让我们使用以下代码找到此链接：

```typescript
this.myAccountLink = page.getByRole("button", { name: " My account" });
```

我们将鼠标悬停在“My account”链接上以打开下拉菜单查看并单击注册链接：

```typescript
async hoverMyAccountLink(): Promise<void> {
    await this.myAccountLink.hover({ force: true });
}
```

**必须找到并点击注册链接才能打开注册页面**。我们可以在Chrome DevTools中注意到registerLink定位器；这个WebElement的作用就是链接：

![](/assets/images/2025/automatedtest/playwrightautomatedendtoendtesting05.png)

以下函数将悬停在MyAccountLink上，当下拉列表打开时，它将找到并单击registerLink：

```typescript
async navigateToRegistrationPage(): Promise<void> {
    await this.hoverMyAccountLink();
    await this.registerLink.click();
}
```

让我们为注册页面创建第二个页面对象类，它将保存执行交互的所有字段和函数：

```typescript
async registerUser(
    firstName: string,
    lastName: string,
    email: string,
    telephoneNumber: string,
    password: string
): Promise<MyAccountPage> {
    await this.firstNameField.fill(firstName);
    await this.lastNameField.fill(lastName);
    await this.emailField.fill(email);
    await this.telephoneField.fill(telephoneNumber);
    await this.passwordField.fill(password);
    await this.confirmPassword.fill(password);
    await this.agreePolicy.click();
    await this.continueBtn.click();

    return new MyAccountPage(this.page);
}
```

我们可以使用getByLabel()函数来定位字段，然后创建registerUser()函数进行交互并完成注册。

让我们创建my-account-page.ts以进行header断言，并更新注册场景的fixture文件。我们将使用navigationToRegistrationPage()访问注册页面并断言注册帐户标题。然后，我们将使用register-user-data.json中的数据从RegistrationPage类调用registerUser()。

注册后，我们将断言检查页眉“Your Account Has Been Created!”是否显示在“My account”页面上。

### 4.5 实现：测试场景2

我们将在第二个测试场景中添加一个产品并验证购物车详细信息是否显示正确的值。

第一个断言检查用户是否已登录，它通过将鼠标悬停在MyAccountLink上并检查菜单中的“Logout”链接是否可见来实现此目的。

现在，我们将使用主页上的搜索框搜索产品。

我们将通过在搜索框中输入值并点击搜索按钮来搜索iPhone，searchForProduct()函数将帮助我们搜索产品并返回SearchResultPage的新实例：

```typescript
const searchResultPage = await homePage.searchForProduct("iPhone");
await searchResultPage.addProductToCart();
```

搜索结果将显示在searchResultPage上。addProductToCart()函数会将鼠标悬停在搜索结果中检索到的页面上的第一个产品上，当鼠标悬停在产品上时，它将单击“Add to Cart”按钮。

将出现一个通知弹出窗口，显示确认文本：

```typescript
await expect(searchResultPage.successMessage).toContainText(
    "Success: You have added iPhone to your shopping cart!"
);
const shoppingCart = await searchResultPage.viewCart();
```

要确认购物车中有商品，首先在弹出窗口上声明确认文本，然后单击viewCart按钮导航到购物车页面。

最后通过一个断言来验证购物车中的产品名称iPhone确认添加了搜索的产品：

```java
await expect(shoppingCart.productName).toContainText("iPhone");
```

### 4.6 测试执行

以下命令将在本地机器上的Google Chrome浏览器上运行测试：

```shell
$ npx playwright test --project=\"Google Chrome\"
```

以下命令将在LambdaTest云网格上的MacOS Sonoma上的最新Google Chrome版本上运行测试：

```shell
$ npx playwright test --project=\"chrome:latest:macOS Sonoma@lambdatest\"
```

让我们更新package.json文件中脚本块中的各个命令：

```json
"scripts": {
    "test_local": "npx playwright test --project=\"Google Chrome\"",
    "test_cloud": "npx playwright test --project=\"chrome:latest:macOS Sonoma@lambdatest\""
}
```

因此，如果我们想在本地运行测试，请运行以下命令：

```shell
$ npm run test_local
```

**要在LambdaTest云网格上运行测试，我们可以运行以下命令**：

```shell
$ npm run test_cloud
```

测试执行完成后，我们可以在LambdaTest Web自动化仪表板和构建详细信息窗口中查看测试结果：

![](/assets/images/2025/automatedtest/playwrightautomatedendtoendtesting06.png)

构建详细信息屏幕提供平台、浏览器名称及其各自的版本、视频、日志、执行的命令以及运行测试所花费的时间等信息。

## 5. 总结

Playwright是一个轻量级且易于使用的测试自动化框架，开发人员和测试人员可以使用多种编程语言轻松配置它。

使用TypeScript的Playwright更加灵活和简单，因为我们不必为配置和设置编写太多样板代码。我们只需要运行一个简单的安装命令，然后立即开始编写测试。