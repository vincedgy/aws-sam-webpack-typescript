## "Webpack to the rescue"

---

> Developing and debugging with SAM & typescript with webpack 

<small>
_ @Vincedgy - 10/2019_
</small>


> "AWS SAM can not use Typescript, 

> and it's quite hard to debug with !"

![](https://s3.amazonaws.com/btoimage/prism-thumbnails/articles/0aa8-2011922-computer-room-1970s-f1257_s1057_it9220.jpg-resize_then_crop-_frame_bg_color_FFF-h_1365-gravity_center-q_70-preserve_ratio_true-w_2048_.jpg)


## Can we do better ?

![](https://media2.giphy.com/media/26u4gfTMlVGZnNKco/source.gif)


# Yes !

with the help of

> ```aws-sam-webpack-plugin```
>_written by Rich Buggy_

[https://www.npmjs.com/package/aws-sam-webpack-plugin](https://www.npmjs.com/package/aws-sam-webpack-plugin)



This presentation is very much inspired 

by Rich Buggy's [video][video]

[video]: https://www.youtube.com/watch?v=my98p7hyaUE

<embed type="video/webm" src="https://www.youtube-nocookie.com/embed/my98p7hyaUE?controls=1&amp;start=407" width="640" height="480" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" />



## ʍɥo ɐɯ ı ?

✌ [https://twitter.com/VinceDgy](https://twitter.com/VinceDgy)

Cloud enabler, CTO, certified AWS Cloud Architect & Dev, 🐳Docker, 🌿Spring , ❤️Angular, ⚛ React and so much more



## Let's do some smart things...

![](https://i.pinimg.com/originals/52/62/93/5262931391db03271654b9607d239ae5.gif)



## Prerequisites


You're located on your linux ou MacOS host

in a directory for sandbox.


You'll need...


NodeJS 10+ and npm installed

![NodeJS](https://philna.sh/assets/posts/node-1305aa9ecfe75c279ce6772534e04dd5999ddd372dcf28ef41c2a9a84b5acdb1.png)


Docker installed (used by SAM)

![Docker](https://i.pinimg.com/originals/a7/ff/ee/a7ffee61eab5dc466a5514a125c3725b.jpg)


- AWS account
- AWS CLI working for this AWS account
- AWS SAM installed

![AWS SAM Local](https://d2908q01vomqb2.cloudfront.net/da4b9237bacccdf19c0760cab7aec4a8359010b0/2017/08/09/aws_sam_introduction-1024x286.png)


Visual Studio Code for Editing

![https://code.visualstudio.com/assets/docs/languages/typescript/Languages_typescript.png](https://code.visualstudio.com/assets/docs/languages/typescript/Languages_typescript.png)


Typescript knowledge (https://github.com/microsoft/TypeScript)

![TypeScript](https://cloudblogs.microsoft.com/uploads/prod/sites/37/2018/03/typescript-banner2.png)



## Create a new application with SAM


Lauch the creation of a sam project for NodeJs 10.x named 'demo'

```shell
$ sam init -r nodejs10.x -n demo

[+] Initializing project structure...

Project generated: ./demo

Steps you can take next within the project folder
===================================================
[*] Invoke Function: sam local invoke HelloWorldFunction --event event.json
[*] Start API Gateway locally: sam local start-api

Read demo/README.md for further instructions

[*] Project initialization is now complete
```


Now open 'demo' project in Visual Studio Code

```shell
cd demo
code .
```


> DEMO
![](http://img.over-blog-kiwi.com/0/86/02/84/20140708/ob_620f9a_mo5.png)


## Install the projet 


Install a package.json in your project for dependencies and install them like so

```shell
npm init -y
```



## Install development dependencies


```shell
$ npm install --save-dev \
  webpack webpack-cli \
  typescript \
  ts-loader \
  aws-sam-webpack-plugin \
  @types/aws-lambda @types/node

[...]

+ aws-sam-webpack-plugin@0.3.1
+ @types/aws-lambda@8.10.33
+ ts-loader@6.2.0
+ @types/node@12.11.1
+ webpack-cli@3.3.9
+ webpack@4.41.2
+ typescript@3.6.4
added 504 packages from 317 contributors and audited 5368 packages in 24.962s
found 0 vulnerabilities
```


- **webpack**: is a module bundler and task runner. Not limited to Web alone ! It handles ES2015+, CommonJS, and AMD module formats. It's now one of the famest module bundler, popularize by `React` and many others.

- **webpack-cli** : the CLI of webpack


- **typescript** : TypeScript is an open-source programming language developed and maintained by Microsoft. It is a strict syntactical superset of JavaScript, and adds optional static typing to the language.

- **ts-loader** : is Typescript loader for webpack. Note : Loaders are transformations that are applied on the source code of a module


- **aws-sam-webpack-plugin** : A Webpack plugin to replace the build step for SAM CLI created by Rich Buggy (https://github.com/SnappyTutorials/aws-sam-webpack-plugin)

- **@types/aws-lambda** : typescript types declaration for aws-lambda
- **@types/node** : typescript types declaration for node



## Install project's dependencies


```shell
npm install aws-sdk source-map-support --save

[...]

+ source-map-support@0.5.13
+ aws-sdk@2.553.0
added 9 packages from 60 contributors, updated 1 package and audited 17654 packages in 6.052s
found 0 vulnerabilities
```


- **aws-sdk** : AWS SDK for Javascript

- **source-map-support** : This module provides source map support for stack traces in node via the V8 stack trace API. It uses the source-map module to replace the paths and line numbers of source-mapped files with their original paths and line numbers. This is THE usefull package for debugging  



## Create a webpack config


Put this **webpack.config.js** into your project

```js
const AwsSamPlugin = require("aws-sam-webpack-plugin");

const awsSamPlugin = new AwsSamPlugin();

module.exports = {
  // Loads the entry object from the AWS::Serverless::Function resources in your
  // template.yaml or template.yml
  entry: awsSamPlugin.entry(),

  // Write the output to the .aws-sam/build folder
  output: {
    filename: "[name]/app.js",
    libraryTarget: "commonjs2",
    path: __dirname + "/.aws-sam/build/"
  },

  // Create source maps
  devtool: "source-map",

  // Resolve .ts and .js extensions
  resolve: {
    extensions: [".ts", ".js"]
  },

  // Target node
  target: "node",

  // Includes the aws-sdk only for development. The node10.x docker image
  // used by SAM CLI Local doens't include it but it's included in the actual
  // Lambda runtime.
  externals: process.env.NODE_ENV === "development" ? [] : ["aws-sdk"],

  // Set the webpack mode
  mode: process.env.NODE_ENV || "production",

  // Add the TypeScript loader
  module: {
    rules: [
      {
        test: /\.tsx?$/,
        loader: "ts-loader"
      }
    ]
  },

  // Add the AWS SAM Webpack plugin
  plugins: [
    awsSamPlugin
  ]
}
```


- Line 8 : is the link between the SAM template's entry for the aws-sam-plugin plugin.
- Line 12 : It will generate a folder with the name of the SAM Function and a ```app.js``` file for each of them
- Line 14 : It will build the JavaScript generated from TypeScript into a folder ```/.aws-sam/build/```
- Line 18 : activate source-map
- Line 31 : you should comment this line (keep prodcution mode activated) 


AWS recommands to **always add the aws-sdk**

even if the SAM CLI don't install it ... 🐵

- Line 37-44 : you add ts-loader included. which compiles TypeScript (*.ts files) to JavaScript for lambda
- Line 47-49 : 'aws-sam-plugin' is activated for webpack 



## Create a Typescript config


Create a typescript.json file on the base directory of your project with the following content :

```json
{
  "compilerOptions": {
    "target": "es2015",
    "module": "commonjs",
    "allowJs": true,
    "checkJs": true,
    "sourceMap": true,
    "esModuleInterop": true
  },
  "include": ["src/**/*.ts", "src/**/*.js"]
}
```



## Add script for building and packaging with webpack


Back to the package.json file add the following line into scripts section :

```json
{
  "scripts": {
    "build": "webpack-cli",
    "watch": "webpack-cli -w",
  }
}

```


You should have a package.json like this (the versions may have changed...):

```json
{
  "name": "demo",
  "version": "1.0.0",
  "description": "This project contains source code and supporting files for a serverless application that you can deploy with the SAM CLI. It includes the following files and folders.",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "build": "webpack-cli",
    "watch": "webpack-cli -w"
  },
  "keywords": [],
  "author": "",
  "license": "ISC",
  "devDependencies": {
    "@types/aws-lambda": "^8.10.33",
    "@types/node": "^12.11.1",
    "aws-sam-webpack-plugin": "^0.3.1",
    "ts-loader": "^6.2.0",
    "typescript": "^3.6.4",
    "webpack": "^4.41.2",
    "webpack-cli": "^3.3.9"
  },
  "dependencies": {
    "aws-sdk": "^2.553.0",
    "source-map-support": "^0.5.13"
  }
}
```



## Create a source folder

![](https://www.sospc20.com/images/liste-commandes-msdos-m.jpg)


Webpack fetch files from within the 'src' folder,

so let's restructure a bit :

- Into the 'demo' directoy create a 'src' directory.
- Then move the 'hello-world' example function created by sam-cli into there.


Within hello-world, you also can remove :

- package.json : useless now
- .npmignore : event more useless after removing package.json file
- tests : because we keep it simple !


You should have something like this : 

```shell
$ tree -I node_modules
.
├── README.md
└── demo
    ├── README.md
    ├── events
    │   └── event.json
    ├── package-lock.json
    ├── package.json
    ├── src
    │   └── hello-world
    │       └── app.js
    ├── template.yaml
    ├── tsconfig.json
    └── webpack.config.js

4 directories, 9 files
```


Now we need to tell SAM that the source is into 'src' 

Edit ```template.yml``` file :

```yaml
#...
Resources:
  HelloWorldFunction:
    Type: AWS::Serverless::Function
    Properties:
      CodeUri: src/hello-world/
      Handler: app.lambdaHandler
      Runtime: nodejs10.x
#...
```



## Build the project 

![https://i.ytimg.com/vi/oZLGpmDvkR8/hqdefault.jpg](https://i.ytimg.com/vi/oZLGpmDvkR8/hqdefault.jpg)


Let's build it !

```shell
$ NODE_ENV=development npm run-script build
```


```shell
$ NODE_ENV=development npm run-script build

> demo@1.0.0 build /Users/vincent/Projects/Yoldee.fr/yoldee-server/demo
> webpack-cli

Hash: 1f2045c517f7f7e8f613
Version: webpack 4.41.2
Time: 112ms
Built at: 10/19/2019 9:35:58 AM
                        Asset      Size              Chunks                   Chunk Names
    HelloWorldFunction/app.js  4.84 KiB  HelloWorldFunction  [emitted]        HelloWorldFunction
HelloWorldFunction/app.js.map  4.85 KiB  HelloWorldFunction  [emitted] [dev]  HelloWorldFunction
Entrypoint HelloWorldFunction = HelloWorldFunction/app.js HelloWorldFunction/app.js.map
[./src/hello-world/app.js] 1.02 KiB {HelloWorldFunction} [built]
```


The build has created : 
- the ```.aws-sam``` folder
- the ```build``` sub-folder
- copy the ```template.yml``` file

```shell
$ tree -a -I node_modules
.
├── README.md
└── demo
    ├── .aws-sam
    │   └── build
    │       ├── HelloWorldFunction
    │       │   ├── app.js
    │       │   └── app.js.map
    │       └── template.yaml
    ├── .gitignore
    ├── .vscode
    │   └── launch.json
    ├── README.md
    ├── events
    │   └── event.json
    ├── package-lock.json
    ├── package.json
    ├── src
    │   └── hello-world
    │       └── app.js
    ├── template.yaml
    ├── tsconfig.json
    └── webpack.config.js

8 directories, 14 files
```


This is the same output if you had executed ```sam build``` command.

> It is AWS SAM compatible.

It also created a ```.vscode``` folder with a ```launch.json``` file that will help for debugging.



## DEBUG it !

![](https://image.spreadshirtmedia.com/image-server/v1/mp/products/T1172A54MPA3563PT17X15Y43D1023302417FS3483/views/1,width=1200,height=630,appearanceId=54,backgroundColor=F2F2F2,modelId=2672,crop=design,version=1565935337,modelImageVersion=1555319047/funny-debugging-process-list-app-programming-womens-vintage-sport-t-shirt.jpg)


The build has created a ```.vscode``` folder with a launch.json 

That specifies the TCP port 5858 for the Visual Studio Code debugger.


### Launch sam with the debugging port 5858 activated


Into the ```src/hello-world/app.js``` place a breakpoint, for instance, on the return instruction. 

```shell
$ sam local invoke -d 5858 -e events/event.json HelloWorldFunction
```

> -d 5858 : is for the debug mode for SAM


```shell
$ sam local invoke -d 5858 -e events/event.json HelloWorldFunction
Invoking app.lambdaHandler (nodejs10.x)
2019-10-19 09:48:08 Found credentials in shared credentials file: ~/.aws/credentials

Fetching lambci/lambda:nodejs10.x Docker container image..........................................................................................................................................................................................................................
Mounting /Users/vincent/Projects/Yoldee.fr/yoldee-server/demo/.aws-sam/build/HelloWorldFunction as /var/task:ro,delegated inside runtime container
Debugger listening on ws://0.0.0.0:5858/e20fb54e-ad3c-4262-acb3-889ac1452373
For help, see: https://nodejs.org/en/docs/inspector
```


At this point SAM is listening to the debugger on port 5858.


### Actualy debug it !

> DEMO
![](http://img.over-blog-kiwi.com/0/86/02/84/20140708/ob_620f9a_mo5.png)


In Visual Studio Code, do clic on the Debugging section and found the right config for your function.

The plugin as created it automatically for you !


You're debugging... 

the breakpoint stops... 

you see the variables... 

# it's working !


```shell
$ sam local invoke -d 5858 -e events/event.json HelloWorldFunction
Invoking app.lambdaHandler (nodejs10.x)
2019-10-19 09:54:39 Found credentials in shared credentials file: ~/.aws/credentials

Fetching lambci/lambda:nodejs10.x Docker container image......
Mounting /Users/vincent/Projects/Yoldee.fr/yoldee-server/demo/.aws-sam/build/HelloWorldFunction as /var/task:ro,delegated inside runtime container
Debugger listening on ws://0.0.0.0:5858/32669918-7ec7-4347-8995-8ccec512e1db
For help, see: https://nodejs.org/en/docs/inspector
Debugger attached.
START RequestId: 8356f349-22cb-145d-623e-19c0d0789765 Version: $LATEST
END RequestId: 8356f349-22cb-145d-623e-19c0d0789765
REPORT RequestId: 8356f349-22cb-145d-623e-19c0d0789765  Init Duration: 42325.95 ms      Duration: 32067.74 ms   Billed Duration: 32100 ms     Memory Size: 128 MB      Max Memory Used: 55 MB
{"statusCode":200,"body":"{\"message\":\"hello world\"}"}
```



## Now let's use Typescript !

![](https://fossbytes.com/wp-content/uploads/2019/07/typescript_redmonk-programming-language.jpg)


### First step 

rename the ```app.js``` file to ```app.ts``` !


Then run webpack in **watch mode** : 

```shell
$ NODE_ENV=development npm run-script watch

> demo@1.0.0 watch /Users/vincent/Projects/Yoldee.fr/yoldee-server/demo
> webpack-cli -w

webpack is watching the files…

Hash: 1cb4bc5dfac53e51147d
Version: webpack 4.41.2
Time: 1640ms
Built at: 10/19/2019 11:23:51 AM
                        Asset     Size              Chunks                   Chunk Names
    HelloWorldFunction/app.js  5.5 KiB  HelloWorldFunction  [emitted]        HelloWorldFunction
HelloWorldFunction/app.js.map  5.1 KiB  HelloWorldFunction  [emitted] [dev]  HelloWorldFunction
Entrypoint HelloWorldFunction = HelloWorldFunction/app.js HelloWorldFunction/app.js.map
[./src/hello-world/app.ts] 1.68 KiB {HelloWorldFunction} [built]

```


Edit ```app.ts``` and define types for our event and context as parameters of our Handler :

```ts

exports.lambdaHandler = async (event:AWSLambda.APIGatewayEvent, 
              context:AWSLambda.APIGatewayEventRequestContext) => {
  let response
  try {
        response = {
            'statusCode': 200, 
            'body': JSON.stringify({ message: 'hello world' })
        }
    } catch (err) {
        console.log(err)
        return err
    }
    return response
}
```


Webpack wakes up and compile ```app.ts```

```shell
$ tree -a -I node_modules                                         
.
├── .aws-sam
│   └── build
│       ├── HelloWorldFunction
│       │   ├── app.js
│       │   └── app.js.map
│       └── template.yaml
├── .gitignore
├── .vscode
│   └── launch.json
├── README.md
├── events
│   └── event.json
├── package-lock.json
├── package.json
├── src
│   └── hello-world
│       └── app.ts
├── template.yaml
├── tsconfig.json
└── webpack.config.js
```


You can check the generated ```app.js``` in ```.aws-sam``` directory.


Now let's launch an invocation of our 'TypeScripted' lambda


```shell
$ sam local invoke -e events/event.json HelloWorldFunction 
Invoking app.lambdaHandler (nodejs10.x)
2019-10-19 11:35:21 Found credentials in shared credentials file: ~/.aws/credentials

Fetching lambci/lambda:nodejs10.x Docker container image......
Mounting /Users/vincent/Projects/Yoldee.fr/yoldee-server/demo/.aws-sam/build/HelloWorldFunction as /var/task:ro,delegated inside runtime container
START RequestId: bd0096cb-c81b-133c-5f4d-c5517c606ddb Version: $LATEST
END RequestId: bd0096cb-c81b-133c-5f4d-c5517c606ddb
REPORT RequestId: bd0096cb-c81b-133c-5f4d-c5517c606ddb  Duration: 18.94 ms      Billed Duration: 100 ms Memory Size: 128 MB     Max Memory Used: 42 MB
{"statusCode":200,"body":"{\"message\":\"hello world\"}"}
```


## Typescript is working !

![](https://media.giphy.com/media/XreQmk7ETCak0/giphy.gif)




## Going further ?


### Debugging with SAM

- [https://www.slideshare.net/AmazonWebServices/serverlessaws-sam-cli-session-developer-meet-up](https://www.slideshare.net/AmazonWebServices/serverlessaws-sam-cli-session-developer-meet-up)
- [https://aws.amazon.com/fr/blogs/opensource/news-may-11-2018/](https://aws.amazon.com/fr/blogs/opensource/news-may-11-2018/)


### TypeScript 

- [https://www.wakefly.com/blog/what-is-typescript-and-why-should-you-use-it/](https://www.wakefly.com/blog/what-is-typescript-and-why-should-you-use-it/)
- [https://www.atyantik.com/setting-up-webpack-with-typescript-part-3-2-the-novice-programmer/](https://www.atyantik.com/setting-up-webpack-with-typescript-part-3-2-the-novice-programmer/)



# Thank you

![](https://static.hitek.fr/img/actualite/2013/07/12/1373633772evolutionofgeek.jpg)

https://github.com/vincedgy/aws-sam-webpack-typescript