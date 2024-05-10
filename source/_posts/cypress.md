---
title: Cypress使用简介
categories:
  - web前端
date: 2024-05-10 17:10:26
tags:
---

[Cypress](https://www.cypress.io/) 是为现代 web 构建的下一代前端测试工具。可以测试所有基于浏览器运行的应用，支持所有类型测试：端到端测试、组件测试、集成测试、单元测试。

## 与Selenium Webdriver、 Puppeteer 的区别

- 大多数测试工具(如Selenium)通过在浏览器外部运行并通过网络执行远程命令来运行，而Cypress与你的应用程序相同的运行循环中执行。 不需要支持进程间通信所需要的控制协议，解决了进程间发送接受命令所带来的响应延迟。

- 由于Cypress在应用程序中运行，这意味着它可以本机访问每个对象。无论它是Window、Document、DOM元素、应用程序实例、函数、计时器、service worker还是其他任何东西——您都可以在Cypress测试中访问它。没有对象序列化，没有在线协议——你可以访问所有的东西。您的测试代码可以访问应用程序代码可以访问的所有相同对象。

- Cypress在运行测试时会截取快照。我们可以将鼠标悬停在spec的每个命令上，以便查看每个步骤都发生了什么。也可以在配置中打开video开关，测试完成后会生成测试录像。

- 可以在测试运行时使用开发人员工具，方便的查看每个控制台消息、每个网络请求。可以检查元素，甚至可以直接调试测试代码。


**缺点** 

- 只支持JS编写测试用例
- 不支持多个浏览器tab
- 一次只能操作一个浏览器实例

## 使用Cypress
以下通过CRA创建的TS模版项目为例。项目地址: https://github.com/niuniuliuliu/cra-cypress


### 安装Cypress，

```
yarn add cypress cross-env mocha --dev
```

### 启动Cypress

```
cypress open --component -b chrome
```

通过--browser(-b)参数可以指定cypress启动时使用的浏览器，如:
```
cypress open --component -b firefox
```

通过--spec参数指定要启动的测试文件,如:
```
cypress run --spec src/__tests__/App.test.tsx --component -b chrome
```

### Cypress配置
首次运行 cypress 时，cypress 会启动一个向导来引导你创建一个配置文件。 创建后的 cypress.config.ts 文件内容如下

```
import { defineConfig } from "cypress";

export default defineConfig({
  component: {
    devServer: {
      framework: "create-react-app",
      bundler: "webpack",
    },
    specPattern: ["src/**/*.test.tsx", "src/**/*.feature"],
  },
});
```
其中specPattern指定cypress测试文件的目录


对于 ts 项目，在 tsconfig.json 中添加 cypress 的配置

```
{
  "compilerOptions": {
    "target": "es5",
    "lib": ["dom", "dom.iterable", "esnext"],
    "types": ["cypress", "node"],
    "allowJs": true,
    "skipLibCheck": true,
    "esModuleInterop": true,
    "allowSyntheticDefaultImports": true,
    "strict": true,
    "forceConsistentCasingInFileNames": true,
    "noFallthroughCasesInSwitch": true,
    "module": "esnext",
    "moduleResolution": "node",
    "resolveJsonModule": true,
    "isolatedModules": true,
    "noEmit": true,
    "jsx": "react-jsx"
  },
  "include": ["src", "cypress"]
}

```

### 组件测试
cypress原生支持React，Angular，Vue，Svelte等框架的组件测试。 由于Cypress基于浏览器运行，在测试中既可以测试组件的功能，也可以测试组件的样式等。在测试中可以非常直观的看到组件的外观。
https://docs.cypress.io/guides/component-testing/overview

一个简单的组件测试例子
```
import React from "react";
import App from "../App";

it("mounts", () => {
  cy.mount(<App />);
  cy.getByTestId("count_button").click();
  cy.getByTestId("count_value").should("contain.text", 1);
});
```

### 端到端测试
通过Cypress我们可以直接访问要测试的应用，进行端到端测试。 这里我们先在本地启动了我们的应用，然后通过Cypress访问该应用，然后模拟用户操作，进行测试。
安装[start-server-and-test包](https://www.npmjs.com/package/start-server-and-test)，我们可以在本地先启动服务然后运行我们的测试
```
yarn add start-server-and-test --dev
```
通过以下命令启动测试
```
start-server-and-test \"yarn start\" \"3000\" \"yarn test:e2e\"
```
一个简单的端到端测试例子
```
describe("E2E Test", () => {
  it("Visit My Local Application", () => {
    cy.visit("http://localhost:3000");

    cy.get('#root').should('be.visible');

    cy.getByTestId("count_button").click();
    cy.getByTestId("count_value").should("contain.text", 1);
  });
});
```

### plugins

#### code coverage
要配置测试覆盖率，需要用到babel-plugin-istanbul这个插件，它兼容Karma和Mocha，而Cypress本身就使用了Mocha的bdd语法。
```
yarn add babel-plugin-istanbul istanbul-lib-coverage nyc --dev
yarn add cypress @cypress/code-coverage --dev
```
添加nyc.config配置文件
```
module.exports = {
  all: true,
  exclude: [
    "src/__tests__/",
    "**/*.d.ts",
    "**/reportWebVitals.ts",
    "**/*.css",
    "**/*.svg",
  ],
  include: ["src"],
  reporter: ["html", "lcov"],
  "report-dir": "coverage",
  "check-coverage": true,
};

```
修改Cypress配置 cypress.config.ts
```
require("@cypress/code-coverage/task")(on, config);
```
配置cypress/support/component.ts
```
import '@cypress/code-coverage/support';
```
修改babel配置，添加istanbul plugin, 如果是CRA创建的项目，webpack babel-loader配置中设置了babelConfig为false，需要运行eject命令来弹出webpack配置，才能使babel.config.js生效
```
module.exports = {
  env: {
    test: {
      plugins: ["istanbul"],
    },
  },
};

```

#### Cucumber
Cucumber 是一个能够理解用普通语言 描述的测试用例的行为驱动开发（BDD）的自动化测试工具。使用该插件可以使用Cucumber的方式来编写测试用例。

安装cypress-cucumber-preprocessor
```
yarn add @badeball/cypress-cucumber-preprocessor @cypress/webpack-dev-server --dev
```
修改Cypress的webpack配置 cypress.config.ts
```
rules: [
              {
                test: /\.feature$/,
                use: [
                  {
                    loader: "@badeball/cypress-cucumber-preprocessor/webpack",
                    options: devServerConfig.cypressConfig,
                  },
                ],
              },
            ],
```

添加.cypress-cucumber-preprocessorrc.json 配置文件，配置stepDefinitions的目录等。
```
{
    "stepDefinitions": [
        "cypress/cucumber/step_definitions/[filepath]/**/*.{tsx,ts}",
        "cypress/cucumber/step_definitions/[filepath].{tsx,ts}",
        "cypress/cucumber/step_definitions/**/*.{tsx,ts}"
    ],
    "nonGlobalStepDefinitions": true,
    "commonPath": "cypress/cucumber/step_definitions/common",
    "nonGlobalStepBaseDir": "cypress/cucumber"
}
```

## 参考
[https://cucumber.io/](https://cucumber.io/)

[https://github.com/badeball/cypress-cucumber-preprocessor/blob/master/docs/readme.md](https://github.com/badeball/cypress-cucumber-preprocessor/blob/master/docs/readme.md)

[https://github.com/istanbuljs/babel-plugin-istanbul](https://github.com/istanbuljs/babel-plugin-istanbul)