# 什么是DevOps

原文：https://docs.microsoft.com/zh-cn/devops/what-is-devops



开发（Dev）和运营（Ops），DevOps 是人员、流程和技术的联合，以不断向客户提供价值。

DevOps 对团队而言意味着什么呢？DevOps 可以实现以前孤立的角色（开发、IT运营、质量工程和安全性），协调和协作，以生成更好的、更可靠的产品。通过采用 DevOps 区域性以及 DevOps 做法和工具，团队可以更好地响应客户需求，提高其生成的应用程序的信心，并更快速地实现业务目标。

## DevOps 和应用程序生命周期

DevOps 影响应用程序生命周期的整个**计划、开发、交付**和**操作**阶段。每个阶段依赖于其他阶段，而阶段并不特定于角色。在真正的 DevOps 区域性中，每个角色都涉及到某个范围。

![devops-lifecycle.png](./images/What_is_devops/devops-lifecycle.png)

### 计划（规划）

在计划阶段，DevOps 团队 ideate、定义和说明它们所构建的应用程序和系统的特性和功能。它们跟踪从单产品任务到跨多个产品组合的任务的级别较低和较高的进度。使用**看板**来创建积压工作（Backlog）、跟踪bug、使用**Scrum**管理敏捷软件开发和使用**仪表板**直观呈现进度是 DevOps 团队计划**灵活性**和**可见性**的一些方式。

### 开发

“开发”阶段包含**编码的所有方面**，即编写代码、测试、检查和团队成员的代码集成，还可以将代码构建为可部署到各个环境中的生成项目。Teams 使用**版本控制**（通常为Git）来协作处理代码和并行工作。他们还设法迅速创新，而不会牺牲质量、稳定性和生产力。为此，它们使用高效的工具，自动执行常见和手动步骤，并通过 **自动化测试** 和 **集成测试** 来循环访问。

### 交付

交付，是指以一致且可靠的方式将应用程序部署到生产环境的过程。理想情况下可通过 **持续交付**。“交付”阶段还包括部署和配置构成这些环境的完全控制基础结构。这些环境通常使用基础设施等技术作为代码（IaC）、**容器**和**微服务**。

DevOps 团队**使用清晰的手动批准阶段来定义发布管理过程**。它们还设置了在阶段之间移动应用程序的自动入口，直到它们可供客服使用。自动执行这些过程可使它们具有可伸缩性、可重复、受控且**经过充分测试**。这样，时间 DevOps 的团队就可以轻松、自信、放心的交付。

### 运营

操作阶段涉及到在生产环境中 维护、监视和故障排除应用程序，这些应用程序通常托管在公共云和混合云中。采用 DevOps 做法时，团队努力确保系统可靠性和高可用性，并瞄准零停机时间，同时增强安全性和管理。

DevOps 团队采用安全的部署实践来确定问题，并在问题发生时迅速降低问题。维护此警惕需要丰富的遥测、可操作的报警，并完全了解应用程序和基础系统。
