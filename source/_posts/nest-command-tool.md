---
title: nest-command-tool
date: 2023-07-09 23:30:53
categories:
- typescript
- nestjs
tags:
- npm
- jest
- javascript
- yargs
- commander
---
### 背景
`nest`官方未提供自定义命令的模块，所以出现很多功能丰富的第三方库，比如的有[nestjs-console](https://github.com/Pop-Code/nestjs-console)(基于[commande](https://github.com/tj/commander.js))、[nestjs-command](https://gitlab.com/aa900031/nestjs-command)(基于[yargs](https://github.com/yargs/yargs))，两者都提供了完善的命令行工具所需配置，引入项目及使用也很方便，但都不支持批量注册命令，每个命令的注册都要提供完整的配置信息，对于某些场景可能不是特别合适(比如偷懒)，所以奔着学习的目的新开一个项目实现缺失的功能

### 开发
整个项目功能非常简单，逻辑也很清晰，主要有以下三点：

1. 遍历出`nest`所有`Module`的`providers`，对使用了指定装饰器的`provider`及其属性做处理
2. 将处理结果组装为合法的`yargs`命令，并且注册至`yargs`
3. 提供`entrypoint`，用于上述步骤1和2及命令的执行

具体实现可以在[nest-command](https://github.com/tashuo/nest-command)项目中查看

### 使用
项目提供了`@Command`、`@Commands`和`@OriginYargsCommand`三个装饰器，分别用于单个命令、多个命令及原生yargs命令的注册.

1. @Command

	直接声明单个命令，可以提供完整的配置，也可使用自动生成的配置，如下所示：

	```typescript
	# ./src/simple.command.ts
	import { Injectable } from ‘@nestjs/common’;
	import { Command } from ‘nestjs-command’;

	@Injectable()
	export class SimpleCommand {
    	@Command({
        	command: ‘simple:test1 <p1>’,
        	describe: ‘simple:test1’,
        	params: [
            	{
                	name: ‘p1’,
                	type: ‘positional’,
                	value: {
                    type: ‘string’
                	}
            	}
        	]
    	})
    	test1(p1: string) {
        	console.log(`test1:${p1}`);
    	}

    	@Command()
    	async test2(p1: number, p2 = ‘abc’) {
        	console.log(`test2:${p1}:${p2}`);
    	}
	}

	```
	
	假如`@Command`未配置params参数，会自动获取方法的参数并注册至yargs，但由于无法获取运行时的参数名所以自动注册的参数名无任何逻辑意义，直接看下面help输出中的*test2*就懂啥意思了，但也算为了偷懒少写代码提供了一条不归路：
	
    ```shell
    $ npm run console:dev --help
    Commands:
        cli simple:test1 <p1>           simple:test1
        cli test2 <number1> [unknown2]  test2(auto generated)
    ```
2. @Commands
        
    ```typescript
    # ./test/multiple.command.ts
    import { Injectable } from '@nestjs/common';
    import { Command, Commands } from '../src/decorators';

    @Injectable()
    @Commands()
    export class MultipleCommand {
        async test3() {
            return new Promise<string>((resolve) => setTimeout(() => resolve('test3'), 0));
        }
    
        test4() {
            return 'test4';
        }
    
        @Command({
            command: 'multiple:test5'
        })
        test5() {
            return 'test5';
        }
    }
    ```

    使用`@Commands`装饰的类会自动将所有的方法注册为独立的command，**便于批量注册**，但由于typescript装饰器只有编译时才会封装`design:*`等metadata信息，动态装饰器无法获取方法的参数，所以**此种方式只适合不需参数的方法**

    
3. @OriginYargsCommand

    具体示例可查看[origin.command.ts](https://github.com/tashuo/nest-command/blob/master/test/original.command.ts)

    使用`@OriginYargsCommand`装饰的类也可以直接注册为command，**便于复用已有的yargs command**，所以必须保证该类继承了`yargs.CommandModule`，是一个标准的yargs command

    为啥会有这样一个feature呢，初衷是想重载`typeorm`的一些内置command，后面发现这些command没有export...先留着吧


### 最后
欢迎使用和交流～