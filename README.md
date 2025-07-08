# Backend Setup

## Resources:

- [TypeScript](https://www.typescriptlang.org/)
- [MongoDb](https://www.mongodb.com/)
- [Mongoose](https://mongoosejs.com/)
- [Express](https://expressjs.com/)
- [DotEnv](https://www.npmjs.com/package/dotenv)
- [TS Node Dev](https://www.npmjs.com/package/ts-node-dev)
- [TypeScript ESlint](https://typescript-eslint.io/getting-started)

### Installation

```npm
npm init -y
```

```ts
npm i -D typescript
```

```ts
tsc --init
```

In `tsconfig.json` file

```ts
rootDir: ./src
outDir: ./dist
```

```ts
npm i mongoose mongodb
```

```ts
npm i express --save-dev @types/expres
```

```npm
npm i cors --save-dev @types/cors
```

```ts
npm i ts-node-dev
```

```ts
npm i express mongoose zod jsonwebtoken cors dotenv
```

```ts
npm i -D ts-node-dev @types/express @types/cors @types/dotenv @types/jsonwebtoken
```

## [TypeScript ESlint](https://typescript-eslint.io/getting-started)

```ts
npm install --save-dev eslint @eslint/js typescript typescript-eslint
```

In `package.json` file

```json
"scripts": {
    "dev": "ts-node-dev --respawn --transpile-only src/server.ts",
    "lint": "npx eslint ./src",
    "lint:fix": "npx eslint . --fix",
    "build": "tsc"
},
```

Run Project

```ts
npm run dev
```

For Build

```ts
tsc
npm run build
```

Create `.env` file

```ts
PORT = 5000;
DATABASE_NAME = <database-name>;
DATABASE_PASS = <database-password>;
```

## [Project Folder Structure]()

### [MVC Pattern]()

```json
library-management-api/
├── src/
│   ├── models/
│   │   ├── book.model.ts
│   │   └── borrow.model.ts
│   ├── controllers/
│   │   ├── book.controller.ts
│   │   └── borrow.controller.ts
│   ├── routes/
│   │   ├── book.route.ts
│   │   └── borrow.route.ts
│   ├── middlewares/
│   │   └── errorHandler.ts
│   ├── config/
│   │   └── database.ts
│   ├── utils/
│   │   └── apiFeatures.ts
│   ├── app.ts
│   └── server.ts
├── package.json
├── tsconfig.json
└── README.md
```

### [Modular Pattern]()

```json
project-root/
├── src/
│   ├── modules/
│   │   ├── mango/
│   │   │   ├── mango.model.ts
│   │   │   ├── mango.interface.ts
│   │   │   ├── mango.controller.ts
│   │   │   ├── mango.route.ts
│   │   ├── order/
│   │   │   ├── order.model.ts
│   │   │   ├── order.interface.ts
│   │   │   ├── order.controller.ts
│   │   │   ├── order.route.ts
│   │   ├── user/
│   │   │   ├── user.model.ts
│   │   │   ├── user.interface.ts
│   │   │   ├── user.controller.ts
│   │   │   ├── user.route.ts
│   │   ├── routes/
│   │   │   ├── index.ts
│   ├── config
│   │   ├── index.ts
│   ├── server.ts
│   ├── app.ts
├── package.json
├── package-lock.json
├── tsconfig.json
├── README.md
└── ...
```

### [Modular Demo]()

### 1. `server.ts` file

```ts
import cors from "cors";
import express from "express";
import mongoose from "mongoose";
import config from "./config";
import routes from "./modules/routes";

const app = express();

app.use(cors());
app.use(express.json());

app.use(routes);

app.get("/", (req, res) => {
  res.send({ success: true, message: "Welcome to the mango server." });
});

async function server() {
  try {
    await mongoose.connect(
      `mongodb+srv://${config.database_name}:${config.database_pass}@cluster0.b6ov8m0.mongodb.net/mango-server?retryWrites=true&w=majority&appName=Cluster0`
    );
    console.log("Connect to the mongodb.");
    app.listen(config.port, () => {
      console.log(`Server Running on port ${config.port}.`);
    });
  } catch (error) {
    console.error(`Server error ${server}`);
  }
}

server();
```

### 1 `server.ts`-(V1)

```ts
/* eslint-disable no-console */
import { Server } from "http";
import mongoose from "mongoose";
import app from "./app";
import { envVars } from "./app/config/env";

let server: Server;

const startServer = async () => {
  try {
    // console.log(envVars.NODE_ENV);
    await mongoose.connect(envVars.DB_URL);

    console.log("Connected to DB!");

    server = app.listen(envVars.PORT, () => {
      console.log(`Server is listening to port ${envVars.PORT}!`);
    });
  } catch (error) {
    console.log(error);
  }
};

startServer();

// Write here all global error handler code
```

### 2. `src/config/index.ts`

```ts
import dotenv from "dotenv";
import path from "path";

dotenv.config({ path: path.join(process.cwd(), ".env") });

export default {
  port: process.env.PORT,
  database_name: process.env.DATABASE_NAME,
  database_pass: process.env.DATABASE_PASS,
};
```

### 3. `src/modules/mango`

#### 3.1 `src/modules/mango/mango.model.ts`

```ts
import { model, Schema } from "mongoose";
import { IMango } from "./mango.interface";

const mangoSchema = new Schema<IMango>(
  {
    name: { type: String, trim: true, required: true },
    variety: { type: String, trim: true, required: true },
    unit: { type: String, enum: ["KG", "TON"], default: "KG", required: true },
    price: { type: Number, min: 0, required: true },
    stock: { type: Number, min: 0, required: true },
    origin: { type: String, default: "Unknown" },
    season: { type: String, enum: ["Summer", "Winter"], required: true },
  },
  {
    timestamps: true,
    versionKey: false,
  }
);

const Mango = model<IMango>("Mango", mangoSchema);

export default Mango;
```

#### 3.2 `src/modules/mango/mango.interface.ts`

```ts
export interface IMango {
  name: string;
  variety: string;
  unit: "KG" | "TON";
  price: number;
  stock: number;
  origin: string;
  season: "Summer" | "Winter";
}
```

#### 3.3 `src/modules/mango/mango.controller.ts`

```ts
import { Request, Response } from "express";
import Mango from "./mango.model";

// Create new mango
const createMango = async (req: Request, res: Response) => {
  try {
    const data = await Mango.create(req.body);

    res.send({
      success: true,
      message: "Mango Creted successfully.",
      data,
    });
  } catch (error) {
    res.send({
      success: false,
      message: "Error Happend",
      error,
    });
  }
};

// Get all Mango
const getMangos = async (req: Request, res: Response) => {
  try {
    const data = await Mango.find();

    res.send({
      success: true,
      message: "Mango getting successfully",
      data,
    });
  } catch (error) {
    res.send({
      success: true,
      message: "Error",
      error,
    });
  }
};

// Get Single mango by using ID
const getMangoById = async (req: Request, res: Response) => {
  try {
    const mangoId = req.params.mangoId;
    const data = await Mango.findById(mangoId);

    res.send({
      success: true,
      message: "Mango getting Successfully",
      data,
    });
  } catch (error) {
    res.send({
      success: false,
      message: "Error",
      error,
    });
  }
};

// Delete mango by id
const deleteMangoById = async (req: Request, res: Response) => {
  const mangoId = req.params.mangoId;

  const data = await Mango.findByIdAndDelete(mangoId);

  res.send({
    success: true,
    message: "Mango deleted Successfully.",
    data,
  });
};

// Update mango data
const updateMango = async (req: Request, res: Response) => {
  try {
    const mangoId = req.params.mangoId;

    const data = await Mango.findByIdAndUpdate(mangoId, req.body, {
      new: true,
      runValidators: true,
    });

    res.send({
      success: true,
      message: "Mango Updated Succcessfully.",
      data,
    });
  } catch (error) {
    res.send({
      success: false,
      message: "Error",
      error,
    });
  }
};

export const mangoController = {
  createMango,
  getMangos,
  getMangoById,
  deleteMangoById,
  updateMango,
};
```

#### 3.4 `src/modules/mango/mango.route.ts`

```ts
import { Router } from "express";
import { mangoController } from "./mango.controller";

const mangoRoute = Router();

mangoRoute.post("/", mangoController.createMango);
mangoRoute.get("/", mangoController.getMangos);
mangoRoute.get("/:mangoId", mangoController.getMangoById);
mangoRoute.delete("/:mangoId", mangoController.deleteMangoById);
mangoRoute.patch("/:mangoId", mangoController.updateMango);

export default mangoRoute;
```

### 4. `src/modules/routes/index.ts`

```ts
import { Router } from "express";
import mangoRoute from "../mango/mango.route";
import orderRoute from "../order/order.route";
import userRoute from "../user/user.route";

const routes = Router();

routes.use("/mango", mangoRoute);
routes.use("/user", userRoute);
routes.use("/order", orderRoute);

export default routes;
```

## [Global Error Handler Code]()

add this error code into the `server.ts` file.

```ts
process.on("SIGTERM", () => {
  console.log("SIGTERM signal recieved... Server sutting down..");

  if (server) {
    server.close(() => {
      process.exit(1);
    });
  }

  process.exit(1);
});
```

```ts
process.on("SIGINT", () => {
  console.log("SIGING signal recieved... Server shutting down..");

  if (server) {
    server.close(() => {
      process.exit(1);
    });
  }

  process.exit(1);
});
```

```ts
process.on("unhandledRejection", (err) => {
  console.log("Unhandled Rejection detected... Server shutting down..", err);

  if (server) {
    server.close(() => {
      process.exit(1);
    });
  }

  process.exit(1);
});
// Unhandler rejection error
// Promise.reject(new Error("I forgot to catch this promise."));
```

```ts
process.on("uncaughtException", (err) => {
  console.log("Uncaught Exception detected... Server sutting down..", err);

  if (server) {
    server.close(() => {
      process.exit(1);
    });
  }

  process.exit(1);
});
// Uncaught Exception Error
// throw new Error("I forgot to handle this local error");
```

## [Vercel Setup]()

### 1. Create `vercel.json` file

```json
{
  "version": 2,
  "builds": [
    {
      "src": "dist/server.js",
      "use": "@vercel/node"
    }
  ],
  "routes": [
    {
      "src": "/(.*)",
      "dest": "dist/server.js"
    }
  ]
}
```

### 2. Install vercel

```npm
npm i -g vercel
```

### 3. Check vercel verson

```npm
vercel -v
```

### 4. Vercel Login

```npm
vercel login
```

```npm
gitHub login -> Continue with GitHub
```

### 5. Deploy Vercel

```
vercel --prod
```

```
1. Set Up and deploy? yes
2. Which scope do you want to deploy to? MdAfsarHossain
3. Link to existing project? no
4. What's your projects name? mango-server
5. In which directory is your doe located? ./

```
