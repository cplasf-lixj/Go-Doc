# 创建应用

## 1. Create React  App

### 1.1 环境要求

* Node: 版本大于10.16
* npm：版本大于5.6

### 1.2 创建APP

````shell
npx create-react-app my-app 	#创建名为my-app的应用
cd my-app
npm start

# 创建会报错，提示Package pangocairo was not found in the pkg-config search path.
# 执行下面命令接口解决
brew install pango
````

## 2. 基础Next.js框架

### 2.1 环境要求

* Node： 版本大于10.13

### 2.2 创建APP

````shell
# 创建nextjs-app 
npx create-next-app nextjs-app --use-npm
cd nextjs-blog
# 运行APP
npm run dev
````

