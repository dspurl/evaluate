#评价
## 说明
- 该插件依赖dsshop项目，而非通用插件
- 支持版本:dsshop v2.0.0及以上
- 已同步版本：dsshop v2.0.0

## 功能介绍
- 支持对商品进行评价，平台可通过后台进行回复
- 支持自动评价功能，默认收货12后天自动好评
- 支持为同一订单多件商品进行评价
- 商品详情页显示评价记录，默认显示2条，在评价列表显示全部，每次加载8条数据
- 支持用户匿名评价
- 支持用户上传图片、填写评价内容、进行星级评分，默认只要进行星级评分和评价内容即可完成评价
- 评价列表显示用户购买的商品规格，展示评价上传的图片，并支持图片预览
- 数据表为灵活设计，即可以支持其它评价，只需要设置`model_type`和`model_id `即可

## 使用说明
#### 一、 下载comment最新版
#### 二、 解压comment到项目plugin目录下
#### 三、 登录dsshop后台，进入插件列表
#### 四、 在线安装（请保持dsshop的目录结构，且在dev下运行，如已部署到线上，请在本地测试环境安装，因涉及文件操作，请不要在正式环境中进行插件操作）
#### 五、 使用说明

##### 管理员南

- 用户评价后，管理员可以通过评价管理对用户的评价内容进行审核，审核通过的才会在前台展示
- 管理员可以把用户评价的内容进行删除，删除后该条评价和回复都会被删除

#### 六、插件示例代码，以下仅供参考，请根据自己的业务自行实现

##### 运维指南

- 插件提供`automatic:evaluate`命令来实现自动评价功能

```php
protected $commands = [
    ...
    AutomaticDelivery::class,
];
protected function schedule(Schedule $schedule){
    ...
    if (config('comment.automaticEvaluateState')) {    //是否开启自动好评
        $schedule->command('automatic:evaluate')->everyMinute();
    }
}
```

##### 开发指南

###### 配置

```shell
# .env
AUTOMATIC_EVALUATE_STATE=true   #是否开启自动评价功能
AUTOMATIC_EVALUATE=12   #多少天后自动好评
```
#### 网站

###### 我的订单添加相关链接

```vue
#client\nuxt-web\mi\pages\user\indent\list.vue
<template>
	...
	<el-tab-pane label="待收货" name="3"></el-tab-pane>
    <el-tab-pane label="待评价" name="10"></el-tab-pane>
	...
	<div class="operation">
		<div>
			...
            <div v-if="item.state === 1" class="button"><el-button :loading="buttonLoading" size="mini" round @click="cancelOrder(item)">取消订单</el-button></div>
            <NuxtLink :to="{ path: '/comment/score', query: { id: item.id }}" v-if="item.state === 10"><div class="button"><el-button type="danger" size="mini" round>立即评价</el-button></div></NuxtLink>
		</div>
	</div>
</template>
```

###### 订单详情增加评价相关代码

```vue
#client\nuxt-web\mi\pages\user\indent\detail.vue
<template>
	...
	<div :class="{on:indent.state === 5 || indent.state === 11}">
        <div class="chunk">交易成功</div>
        <div class="name"></div>
    </div>
</template>
```
###### 商品详情添加评价相关代码

```js
#client\nuxt-web\mi\pages\product\js\detail.js
import Comment from '@/pages/comment/list'\
import {good} from '@/api/comment'
export default {
  components: {
    ...
    Comment
  },
  data() {
    return {
        commentTotal: 0
    }
  },
  mounted() {
      this.getCommentTotal()
  },
  methods: {
    // 获取评价总数
    getCommentTotal(){
      good({
        limit: 1,
        page: 1,
        good_id:$nuxt.$route.query.id,
        sort:'-created_at'
      }).then(response => {
        this.commentTotal = response.total
      })
    }
  }
}
```
```vue
#client\nuxt-web\mi\pages\product\detail.vue
<template>
	...
	<div class="product-box">
      <div class="tab">
        <span :class="{on:tab === 1}" @click="cutTab(1)">商品详情</span>
        <el-divider direction="vertical"></el-divider>
        <span :class="{on:tab === 2}" @click="cutTab(2)">评价({{commentTotal}})</span>
      </div>
      <div class="detail-box">
        <div class="container" v-loading="tabLoading">
          <div v-if="tab === 1" v-html="goodDetail.details"></div>
          <div v-else-if="tab === 2">
            <Comment></Comment>
          </div>
        </div>
      </div>
    </div>
</template>
```



#### 移动端

###### pages.json添加路由

```json
#client\uni-app\mix-mall\pages.json
,{
    "path": "pages/comment/score",
    "style": {
        "navigationBarTitleText": "评价",
        "app-plus": {
            "bounce": "none"
        }
    }
}
, {
    "path": "pages/comment/list",
    "style": {
        "navigationBarTitleText": "评价列表",
        "enablePullDownRefresh": true,
        "onReachBottomDistance": 50
    }
}
```

###### 个人中心增加评价链接及待评价数量
```vue
#client\uni-app\mix-mall\pages\user\user.vue
<template>
	<!-- 订单 -->
	<view class="order-section">
        ...
        <view class="order-item" @click="navTo('/pages/indent/list?state=4')" hover-class="common-hover"  :hover-stay-time="50">
            <text class="yticon icon-yishouhuo"><text v-if="quantity.remainEvaluated" class="cu-tag badge">{{quantity.remainEvaluated}}</text></text>
            <text>待评价</text>
        </view>
    </view>
</template>
<script>
export default {
    data() {
		return {
			quantity: {
                all: 0,
                obligation: 0,
                waitdeliver: 0,
                waitforreceiving: 0
            },
		};
	},
    onShow(){
        if(this.hasLogin){
            this.getUser()
            this.browse()
            this.noticeConut()
            this.getQuantity()
            this.getUserCouponCount()
        } else {
            this.browseList = []
            this.user = {}
            this.noticeNumber = null
            this.quantity = {
                all: 0,
                obligation: 0,
                waitdeliver: 0,
                waitforreceiving: 0
            }
        }

    },
}
</script>
```

###### 订单列表增加评价按钮

```vue
#client\uni-app\mix-mall\pages\indent\list.vue
<template>
	<view class="action-box b-t">
        ...
        <block v-if="item.state === 10">
        	<button class="action-btn recom" @tap="goScore(item)">立即评价</button>
        </block>
    </view>
</template>
<script>
export default {
    data() {
		return {
			navList: [
                ...
            	{
                    state: 10,
                    text: '待评价',
                    loadingType: 'more',
                    orderList: []
            	}
            ],
		};
	},
    methods: {
        // 评价
        goScore(item){
            uni.navigateTo({
                url: `/pages/comment/score?id=${item.id}`
            })
        },
        //评价成功后回调
        refreshOderList(){
            // 需要重新加载
            this.loadData()
        }
    }
}
</script>
```
###### 订单详情增加评价按钮
```vue
#client\uni-app\mix-mall\pages\indent\detail.vue
<template>
	<!-- 底部 -->
	<view v-if="indentList.state === 1 || indentList.state === 3 || indentList.state === 10" class="footer">
        <view class="price-content"></view>
        <navigator v-if="indentList.state === 1" :url="'/pages/money/pay?id=' + indentList.id" hover-class="none" class="submit">立即支付</navigator>
        <view v-else-if="indentList.state === 3" class="submit" @click="confirmReceipt(indentList)">确认收货</view>
        <view v-else-if="indentList.state === 10" class="submit" @click="goScore(indentList)">立即评价</view>
    </view>
</template>
<script>
export default {
    methods: {
        // 评价
        goScore(item){
            uni.navigateTo({
                url: `/pages/comment/score?id=${item.id}`
            })
        },
        //评价成功后回调
        refreshOderList(){
            // 需要重新加载
            this.getList()
        }
    }
}
</script>
```
###### 添加后台订单进度对评价的支持

```vue
#admin\vue2\element-admin-v3\src\views\IndentManagement\Indent\components\Detail.vue
<template>
</template>
<script>
export default {
    data() {
		return {
		};
	},
    methods: {
		getList() {
            case 5:
            case 11:
              this.order_progress = 4
              break
        }
    }
}
</script>
```
###### 添加订单评价相关状态

```php
#\app\Models\v1\GoodIndent.php
const GOOD_INDENT_STATE_EVALUATE = 10; //状态：待评价
const GOOD_INDENT_STATE_HAVE_EVALUATION = 11; //状态：已评价
const GOOD_INDENT_IS_AUTOMATIC_EVALUATE_YES = 1; //自动好评：是
const GOOD_INDENT_IS_AUTOMATIC_EVALUATE_NO = 0; //自动好评：否
public function getStateShowAttribute()
{
   ...
	case static::GOOD_INDENT_STATE_REFUND_FAILURE:
    	$return = '退款失败';
    	break;
    case static::GOOD_INDENT_STATE_EVALUATE:
        $return = '待评价';
        break;
    case static::GOOD_INDENT_STATE_HAVE_EVALUATION:
    	$return = '已评价';
    	break;
}
```
###### 添加商品评价关联

```php
#api\app\Models\v1\GoodIndentCommodity.php
/**
  * 获取评价
  */
public function Comment(){
    return $this->morphOne('App\Models\v1\Comment', 'model');
}
```
###### 增加评价统计代码

```php
#api\app\Http\Controllers\v1\Client\GoodIndentController.php
public function quantity()
{
    ...
    'waitforreceiving' => 0, //待收货
    'remainEvaluated' => 0, //待评价
    ...
    } else if ($indent->state == GoodIndent::GOOD_INDENT_STATE_TAKE) {
    	$return['waitforreceiving'] += 1;
	} else if ($indent->state == GoodIndent::GOOD_INDENT_STATE_EVALUATE) {
    	$return['remainEvaluated'] += 1;
	}
}
```

###### 添加商品评价记录

```vue
#client\uni-app\mix-mall\pages\product\detail.vue
<template>
		<!-- 评价 -->
		<view class="eva-section">
			<view class="e-header">
				<text class="tit">评价</text>
				<text>({{commentTotal}})</text>
				<navigator hover-class="none" class="tip" :url="'comment?id='+ id">查看全部</navigator>
				<text class="yticon icon-you"></text>
			</view>
			<view class="eva-box" v-for="(item,index) in commentList" :key="index">
				<image class="portrait" :src="item.comment.portrait || '/static/missing-face.png'"  mode="aspectFill" lazy-load></image>
				<view class="right">
					<text class="name">{{item.comment.name}}</text>
					<text class="con">{{item.comment.details}}</text>
					<view class="bot">
						<text class="attr">购买类型：<span v-for="(ite,ind) in item.good_sku.product_sku" :key="ind" class="padding-right-xs">{{ite.value}}</span></text>
						<text class="time">{{item.comment.created_at.split(' ')[0]}}</text>
					</view>
				</view>
			</view>
		</view>
</template>
<script>
import {good as commentGood} from '@/api/comment'
export default {
    data() {
		return {
			...
			commentList: [],
			commentTotal:0
		};
	},
    async onLoad(options) {
		if (id) {
			...
			this.goodEvaluate()
		}
	},
    methods: {
		// 获取评价列表
		commentGood({
            limit: 2,
            page: 1,
            good_id:this.id,
            sort:'-created_at'
        },function(res){
            if(res.total>0){
                this.commentList = res.data
                this.commentTotal = res.total
            }
        })
    }
}
</script>
```

配置模板通知

```php
#api\config\notification.php
'wechat'=>[ //微信公众号
    ...
    'order_evaluate'=>env('WECHAT_SUBSCRIPTION_INFORMATION_ORDER_EVALUATE',''),  //订单评价提醒
    'admin_order_evaluate'=>env('WECHAT_SUBSCRIPTION_INFORMATION_ADMIN_ORDER_EVALUATE',''),  //用户评价通知
    ],
```

```shell
#.env
WECHAT_SUBSCRIPTION_INFORMATION_ORDER_EVALUATE=
WECHAT_SUBSCRIPTION_INFORMATION_ADMIN_ORDER_EVALUATE=
```



## 如何更新插件
- 将最新版的插件下载，并替换老的插件，后台可一键升级
## 如何卸载插件
- 后台可一键卸载
