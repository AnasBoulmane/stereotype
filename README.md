# @express.ts/stereotype
 A simple framework for building efficient and scalable server-side applications, heavily inspired by <a href="https://spring.io" target="blank">Spring MVC</a>.

app.module.ts
```typescript
import {
  ComponentScan,
  ExpressBootApplication,
  IExpressApplication,
  Middleware
} from "@express/stereotype";

const config = require("../config/appConfig.json");

// Create Express server
@ComponentScan([
  path.join(__dirname, "./services/impl"),
  path.join(__dirname, "./controllers"),
])
@ExpressBootApplication({
  settings: [{
    key: "async", value: false }, {
    key: "port", value: process.env.PORT || 3000 }, {
    key: "httpPort", value: process.env.PORT || 3000 }, {
    key: "createForkWorkers", value: true }, {
    key: "views", value: path.join(__dirname, "../views") }, {
    key: "view engine", value: "pug"
  }],
  properties: [
    config
  ],
  middlewares: [
    compression(),
    lusca.xssProtection(true),
    express.static(path.join(__dirname, "public"), { maxAge: 31557600000 })
  ]
})
export class AppModule implements IExpressApplication {
  public expressApp: core.Express;
  public startServers: () => Promise<any>;
  public bootstrap: () => Promise<any>;

  createConnection () {
    // Connect to MongoDB
    // createConnection()
    //   .then(connection => console.log("Connect to MongoDB: ", connection.isConnected))
    //   .catch(errors => console.log(errors));
  }

  public $OnInit () {
    console.log(`$OnInit worker ${process.pid}`);
  }

  public $OnReady () {
    console.log(`$OnReady worker ${process.pid}`);
  }

  @Middleware
  setLocalsUser (req: any, res: any, next: any) {
    res.locals.user = req.user;
    next();
  }

  @Middleware
  goToDistanition (req: any, res: any, next: any) {
    // After successful login, redirect back to the intended page
    if (!req.user &&
      req.path !== "/login" &&
      req.path !== "/signup" &&
      !req.path.match(/^\/auth/) &&
      !req.path.match(/\./)) {
      req.session.returnTo = req.path;
    } else if (req.user &&
      req.path == "/account") {
      req.session.returnTo = req.path;
    }
    next();
  }
}

export const appModule = new AppModule();

export default appModule;

```
server.ts
```typescript
import errorHandler from "errorhandler";

import appModule from "./app.module";

const { expressApp } = appModule;

appModule
  .bootstrap()
  .then((msg: any) => {

    console.info(msg);

    /**
     * Error Handler. Provides full stack - remove for production
     */
    expressApp.use(errorHandler());

    /**
     * Start Express server.
     */
    appModule.startServers()
      .then(msg => console.log(msg))
      .catch(err => console.error(err));
  });

export default appModule;
```
./controllers/UserController.ts
```typescript
import { GetMapping, PostMapping, RequestMapping, RequestMethod } from "@express/router";
import { Autowired, Controller, PathVariable, Request, Response } from "@express/stereotype";

import IMailService from "../services/IMailService";
import IMetierService from "../services/IMetierService";


@Controller()
export default class UserController {

  @Autowired()
  protected mailService: IMailService;

  @Autowired()
  protected metierService: IMetierService;

  /**
   * GET /login
   * Login page.
   */
  @GetMapping("/login")
  getLogin (@Request() req: any,
            @Response() res: any) {
    console.log("getLogin(): ========>", this.metierService.getTemperature());
    if (req.user) {
      return res.redirect("/");
    }
    res.render("account/login", {
      title: "Login"
    });
  }
}  
```
./services/IMetierService.ts
```typescript

export default interface IMetierService {
  getTemperature (): number;
}
```
./services/impl/MetierService.ts
```typescript

import { Service } from "@express/stereotype";
import IMetierService from "../IMetierService";


@Service()
export default class MetierService implements IMetierService {

  getTemperature(): number {
    return Date.now();
  }

}
```
