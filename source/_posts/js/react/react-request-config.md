---
title: react使用request请求数据
date: 2019-01-31 01:14:31
thumbnail: 
categories:
    - js
tags:
    - react
---

## react使用request配置拦截统一错误处理

## 什么是request
[https://github.com/umijs/umi-request](官方文档)
看了官网应该什么都明白了，他是一个替代axios的ajax方案，接下来记录下自己做项目喜欢使用的配置模板；

## 个人喜好配置
``` javascript 
import qs from 'qs';
import router from 'umi/router';
import { extend } from 'umi-request';
import { notification } from 'antd';
import configs from '../../config/env'

const codeMessage = {
  200: '服务器成功返回请求的数据。',
  201: '新建或修改数据成功。',
  202: '一个请求已经进入后台排队（异步任务）。',
  204: '删除数据成功。',
  400: '发出的请求有错误，服务器没有进行新建或修改数据的操作。',
  401: '用户没有权限（令牌、用户名、密码错误）。',
  403: '用户得到授权，但是访问是被禁止的。',
  404: '发出的请求针对的是不存在的记录，服务器没有进行操作。',
  406: '请求的格式不可得。',
  410: '请求的资源被永久删除，且不会再得到的。',
  422: '当创建一个对象时，发生一个验证错误。',
  500: '服务器发生错误，请检查服务器。',
  502: '网关错误。',
  503: '服务不可用，服务器暂时过载或维护。',
  504: '网关超时。',
};

/**
 * 异常处理程序 这里使用官方的处理没有访问到服务器的情况，因为请求到了服务器即使服务器报错 httpStatus 也会是* 200，错误将会封装在json中返回，所以不会进入错误处理
 */
const errorHandler = (error: { response: Response }): Response => {
  const { response } = error;
  if (response && response.status) {
    const errorText = codeMessage[response.status] || response.statusText;
    const { status } = response;

    notification.error({
      message: `请求错误 ${status}`,
      description: errorText,
    });
  } else if (!response) {
    notification.error({
      description: '您的网络发生异常，无法连接服务器',
      message: '网络异常',
    });
  }
  return response;
};

/**
 * 配置request请求时的默认参数 相当于构造函数
 */
const request = extend({
  errorHandler, // 默认错误处理
  credentials: 'include', // 默认请求是否带上cookie
});

// 拦截请求，格式化参数， 设置host，这里的env.API_ENV的取值是环境变量，按环境选择url地址
// 后面的请求中如果配置中携带了 ‘form’ 则是文件，不设置请求头，其他的请求都粉装成form-urlencode来完成；
request.interceptors.request.use((url, options) => ({
  // @ts-ignore
  url: configs[process.env.API_ENV].API_SERVER.concat(url),
  options: {
    ...options,
    headers:
      options.requestType === 'form'
        ? undefined
        : { 'Content-type': 'application/x-www-form-urlencoded' },
    data: options.requestType === 'form' ? options.data.formData : qs.stringify(options.data),
  },
}));

// 拦截返回参数 如果返回401，跳转登陆，这里做了同步处理，因为一些错误是在返回的json中的，如果不是的话可以不用
request.interceptors.response.use(async response => {
  if (response.status === 401) {
    router.push('/user/login');
  }
  const data = await response.clone().json();
  if(!data.success) {
    notification.error({
      description: "请求失败",
      message: data.message,
    });
  }
  return response;
});

export default request;
```

以上是个人使用的一些配置，更多配置可以参考官方文档！
