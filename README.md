# NestJs

1. node 版的 angular
2. 底层基于 express (就想用 koa2 怎么办 ??)
3. typescript

## 设计模式

* 控制反转（Inversion of Control，IoC）
  * 将依赖对象的创建和绑定转移到被依赖对象类的外部来实现
  * 好处：去耦合
  * 实现手段
    * 依赖注入 DI
    * 装饰器
      * constructor: Reflect.getMetadata
      * params （早于 constructor 装饰器执行）:Reflect.defineMetadata

## 依赖注入

### 1. controller

#### http service

> 内部默认使用 express 框架处理请求

* @Controller('user')
* @Req() request: Request
  * 注入 express requset 对象
* @Res() res: Response
  * res.send()
  * res.json()
* @Params('id') id: number
  * 请求路径上动态配置：`@Get('products/:id')`
  * @Param() params
    * params.id
* @Get('hello-world')
  * @Get('ab*cd') 通配符写法
  * @Query('google') google: number
  * @Query() query
* @Post('find-one')
  * @Body() body: any
* @Header('Cache-Control', 'none')
* @HttpCode(204)
* @Redirect('https://nestjs.com', 302)

#### 响应请求

1. return
   * value
   * Promise (async)
   * Observable
     * `Nest` 将自动订阅下面的源并获取**最后**发出的值
2. @Res() res: Response
   * res.status(HttpStatus.CREATED).send();
   * res.status(HttpStatus.OK).json([]);

```ts
import { Controller, Get, Query, Post, Body, Put, Param, Delete } from '@nestjs/common';
import { CreateCatDto, UpdateCatDto, ListAllEntities } from './dto';

@Controller('cats')
export class CatsController {
  @Post()
  create(@Body() createCatDto: CreateCatDto) {
    return 'This action adds a new cat';
  }

  @Get()
  findAll(@Query() query: ListAllEntities) {
    return `This action returns all cats (limit: ${query.limit} items)`;
  }

  @Get(':id')
  findOne(@Param('id') id: string) {
    return `This action returns a #${id} cat`;
  }

  @Put(':id')
  update(@Param('id') id: string, @Body() updateCatDto: UpdateCatDto) {
    return `This action updates a #${id} cat`;
  }

  @Delete(':id')
  remove(@Param('id') id: string) {
    return `This action removes a #${id} cat`;
  }
}
```

#### DTO

* (数据传输对象) 模式
* xx.dto.ts
* models interface ts

### 2. Provider (service)

> Provider只是一个用 `@Injectable()` 装饰器注释的类

* @Injectable()
* <font color="red">@inject() </font>
  * `@Optional() @Inject('HTTP_OPTIONS') private readonly httpClient: T`

### 3. Module 模块

* @Global()
  * 装饰器使模块成为全局作用域。 全局模块应该只注册一次，最好由根或核心模块注册
  * 不需要在 `imports` 数组中导入  @Global 的模块
* providers
* controllers
* imports
* exports
  * 默认情况下，模块是**单例**，因此您可以轻松地在多个模块之间共享**同一个**提供者实例
  * exports services
  * 每个导入 `CatsModule` 的模块都可以访问 `CatsService` ，并且它们将共享相同的 `CatsService` 实例

```ts
// cats.module.ts
import { Module } from '@nestjs/common';
import { CatsController } from './cats.controller';
import { CatsService } from './cats.service';

@Module({
  controllers: [CatsController],
  providers: [CatsService],
  exports: [CatsService],
})
export class CatsModule {}

// app.module.ts
import { Module } from '@nestjs/common';
import { CatsModule } from './cats/cats.module';

@Module({
  imports: [CatsModule],
})
export class ApplicationModule {}
```

## 中间件

* next

```ts
// koa 的 next 洋葱模型
function componse(middlewares) {
    // componse(middlewares)(ctx) 被 http.createServer 调用 返回 promise
    // 是一个不断往上把 middleware 构建成 next 的过程
   // middleware 一定是返回 Promise / async
    return async ctx => {
        const createNext = (middleware, next) => () => middleware(ctx, next);
        // 这个 next 是最后一个 next
        let next = () => Promise.resolve();
        // 从最后一个中间件开始计算出第一个中间件
        for (let i = middlewares.length - 1; i >= 0; i--) {
            const middleware = middlewares[i];
            next = createNext(middleware, next);
        }
        // 这里得到的 next 是嵌套好参数的第一个中间件
        // 中间件实参已通过闭包方式传递
        await next();
    }
}

class App {
    constructor() {
        this.ctx = {};
        this.middlewares = [];
    }
    use(middleware) {
        this.middlewares.push(middleware);
    }
    listen() {
        componse(this.middlewares)(this.ctx);
    }
}
```

```ts
// demo run...
const app = new App();

app.use(async (_ctx, next) => {
    console.log('1 start');
    console.log(next.toString());
    await next(); // 此 next 表示 下一个中间件 next = () => nextMiddleware(_ctx, _next)
    console.log('1 end');
});

app.use(async (_ctx, next) => {
    console.log('2 start');
    console.log(next.toString());
    await next();
    console.log('2 end');
});

app.listen();
```

## 拦截器

* @Injectable()
  * implements NestInterceptor
* @UseInterceptors(new LoggingInterceptor())
  * 导入 @Injectable() 装饰的 class

## 管道

* @UsePipes(new JoiValidationPipe(createCatSchema))
  * 具有 @Injectable() 装饰器的类。管道应实现 PipeTransform 接口
