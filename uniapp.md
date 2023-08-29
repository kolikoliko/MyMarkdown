# uniapp开发



本文章用于学习小程序搭配uniapp的记录



## tabbar

先新建页面，然后再page.json里面设置

```js
"tabBar": {
		"list": [{
				"pagePath": "pages/home/home",
				"text": "首页",
				"iconPath": "static/icon/鸡腿.svg",
				"selectedIconPath": "static/icon/鸡腿.svg"
			}, {
				"pagePath": "pages/cate/cate",
				"text": "购物车",
				"iconPath": "static/icon/三明治.svg",
				"selectedIconPath": "static/icon/三明治.svg"
			}, {
				"pagePath": "pages/cart/cart",
				"text": "探索",
				"iconPath": "static/icon/鸡尾酒.svg",
				"selectedIconPath": "static/icon/鸡尾酒.svg"
			}, {
				"pagePath": "pages/my/my",
				"text": "我的",
				"iconPath": "static/icon/煎蛋.svg",
				"selectedIconPath": "static/icon/煎蛋.svg"
			}
		]
	}
```



## 配置小程序的请求

安装npm包

```shell
npm install @escook/request-miniprogram
```

配置路径，在```main.js```中加入如下

```shell
import { $http } from '@escook/request-miniprogram'

uni.$http=$http
```

