## 安装插件

```
npm i vite-plugin-compression2@0.11.0 -D
```

## 添加配置

在vite.config.ts文件中添加下面的配置：

```ts
import {compression} from 'vite-plugin-compression2'

export default defineConfig((options) => {
  return {
    base: '/hdsp-dataservice/',
    plugins: [
      vue(),
      ...
      compression({
        threshold:2000, // 设置只有超过 2k 的文件才执行压缩
        deleteOriginalAssets:false, // 设置是否删除原文件
        skipIfLargerOrEqual:true, // 如果压缩后的文件大小与原文件大小一致或者更大时，不进行压缩
      // 其他的属性暂不需要配置，使用默认即可
      }),
    ],
  }
});
```

## 修改nginx配置，开启gzip

```nginx
    location /hdsp-dataservice {
        gzip_static on;
        alias  /home/hdsp/hdsp-dataservice-static/;
        index  index.html;
        try_files $uri $uri/ /hdsp-dataservice/index.html;
    }
```