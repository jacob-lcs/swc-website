# 从 tsc 迁移

如果你正在从 TypeScript 编译器(tsc)迁移，有几点需要注意。

## TypeScript 版本

SWC 支持最新稳定版的 TypeScript。

## isolatedModules: true

SWC 是逐文件工作的，所以任何依赖于理解完整类型系统的代码转换将无法工作。

如果你遇到这些限制，某些 TypeScript 特性如 const enums 和命名空间可能会导致运行时问题。

在这种情况下，在 TypeScript 中使用 [`isolatedModules`][isolatedmodules] 标志可以帮助你警告任何可能无法被 SWC 正确解释的代码。

有关更多详情，请参阅[相关问题的评论](https://github.com/swc-project/swc/issues/7101#issuecomment-1480610668)。

[isolatedmodules]: https://www.typescriptlang.org/tsconfig#isolatedModules

## importsNotUsedAsValues: "error"

由于上述原因，SWC 无法完全辨别导入的绑定是`值`还是`类型`。

将 [`importsNotUsedAsValues`][importsnotusedasvalues] 选项设置为 `error` 将确保 TypeScript 在类型检查期间正确地将所有类型导入标记为 `type`，从而在 SWC 中准确地删除它们。

[importsnotusedasvalues]: https://www.typescriptlang.org/tsconfig#importsNotUsedAsValues

## esModuleInterop: true

TypeScript 的导入互操作性偏离了 ES6 模块规范。

而 SWC 则采用了类似于 Babel 的方法（有时可能更为严格）。

启用 [esModuleInterop][esmoduleinterop] 选项可确保 tsc 的行为与 SWC 的行为保持一致。

[esmoduleinterop]: https://www.typescriptlang.org/tsconfig#esModuleInterop

## verbatimModuleSyntax: true

这是 TypeScript 5.0 中引入的新选项，用于替代 `isolatedModules`、`preserveValueImports` 和 `importsNotUsedAsValues`。
请查看[发布说明][verbatimmodulesyntax]了解更多详情。

[verbatimmodulesyntax]: https://devblogs.microsoft.com/typescript/announcing-typescript-5-0-beta/#verbatimmodulesyntax

## useDefineForClassFields

此问题涉及 `[[Define]]` 和 `[[Set]]` 的语义。

谁不需要关注它？

- 从不使用类的人。
- 使用类但从不使用继承的人。

谁需要特别注意这个问题？

- 装饰器用户。

如果在你的 `tsconfig.json` 中已经设置了该值，那么相同的值可以在 swc 的配置中使用。

如果尚未设置并且你遇到问题，则有必要补充此设置。

应当注意，此选项的默认值将根据你的 `tsconfig.json` 的 `target` 而变化。

请查看 [useDefineForClassFields][usedefineforclassfields] 选项。

> 如果 target 是 ES2022 或更高（包括 ESNext），则为 true，否则为 false。

[usedefineforclassfields]: https://www.typescriptlang.org/tsconfig#useDefineForClassFields

## 已知问题

- [TypeScript#16166](https://github.com/microsoft/TypeScript/issues/16166)
  ES6 导入不会被 tsc 提升。如果你依赖于 tsc 的错误实现，在迁移到 swc 时可能会遇到问题，因为 swc 更严格地保留 ES 模块语义。

## 注意事项

SWC 只转译代码，不执行类型检查。
因此，建议你继续使用 tsc 来检测任何类型错误。
