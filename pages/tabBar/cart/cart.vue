<template>
	<view>
		<!-- 购物列表 -->
		<view class="goods-list">
			<view class="empty" v-if="goodsList.length==0">
				购物车空空如也
			</view>
			<view class="row" v-for="(item,index) in  goodsList" :key="index" >
				<!-- 删除按钮 -->
				<view class="menu" @tap="handleSingleDelete(item)">
					<view class="icon iconfont">
						&#xe6a6;
					</view>
				</view>
				<!-- 商品 -->
				<view class="production" 
				@touchstart="handleTouchStart(index,$event)"
				@touchmove="handleTouchMove(index,$event)"
				@touchend="handleTouchEnd(index,$event)"
				:class="[theIndex==index?'open':'close']">
					<!-- checkbox -->
					<view class="container" @tap="handleCheckbox(item)">
						<view class="checkbox">
							<view :class="{'on':item.selected}"></view>
						</view>
					</view>
					<!-- 商品详情 -->
					<view class="goods-info" @tap="handleGoodsInfo(item)">
						<view class="img">
							<image :src="item.img" mode=""></image>
						</view>
						<view class="info">
							<view class="title">
								{{item.name}}
							</view>
							<view class="spec">
								{{item.spec}}
							</view>
							<view class="price-number">
								<view class="price">
									¥{{item.price}}
								</view>
								<counter :goodsInfo="item" @add="add(item)" @sub="sub(item)"/>
							</view>
						</view>
					</view>
				</view>
			</view>
		</view>
		<!-- 底部菜单 -->
		<view class="footer" v-if="goodsList.length>0" :style="{bottom:footerBottom}">
			<!-- checkbox -->
			<view class="container" @tap="handleSelectAll">
				<view class="checkbox">
					<view :class="{'on':selectAll}"></view>
				</view>
			</view>
			<view class="delBtn" v-if="selectList.length>0" @tap="handleMulDelete">
				删除
			</view>
			<view class="settlement">
				<view class="sum">
					合计：
					<view class="money">
						¥{{sumMoney}}
					</view>
				</view>
				<view class="btn" @tap="handleConfirm">
					结算{{selectList.length}}
				</view>
			</view>
		</view>
	</view>
</template>

<script>
	import counter from '../../../components/counter.vue'
	export default{
		components:{
			counter
		},
		data(){
			return{
				goodsList:[],
				theIndex:null,//控制滑动效果 当前滑动下标
				oldIndex:null,//上一个滑动下标
				initXY:[],
				footerBottom:0,
				selectList:[],
				selectAll:false
				
			}
		},
		computed:{
			sumMoney:{
				get(){
					let money=0;
					this.selectList.forEach((item)=>{
						money += item.price*item.number
					})
					return money.toFixed(2);
				},
				set(){
					
				}
			}
		},
		onShow(){
			uni.getStorage({
				key:"goodsList",
				success:(res=>{
					// 将所有商品的选中状态都设为false
					res.data.forEach((item)=>{
						item.selected=false;
					})
					this.goodsList=res.data;
					// console.log(this.goodsList)
					// 属性数据的初始化
					this.selectList=[];
					this.selectAll=false;
					this.sumMoney='0.00';
				})
			})
		},
		onLoad() {
			//兼容h5下的底部菜单
			//#ifdef H5
			this.footerBottom=document.getElementsByTagName("uni-tabbar")[0].offsetHeight+'px';
			//#endif
		},
		methods:{
			handleCheckbox(item){
				item.selected = !item.selected;
				//如果是选中状态push到selectList里
				let isExist=this.selectList.indexOf(item);
				if(isExist>-1){
					// 数组中是否已包含该item
					this.selectList.splice(isExist,1);
				}else{
					this.selectList.push(item);
				}
				
				// 全选状态
				if(this.selectList.length==this.goodsList.length){
					this.selectAll=true;
				}else{
					this.selectAll=false;
				}
			},
			handleGoodsInfo(item){
				uni.navigateTo({
					url:"../../goods/goods?goodsInfo="+JSON.stringify(item)
				})
			},
			handleTouchStart(index,event){
				// 多点触控 不触发
				if(event.touches.length>1){
					return 
				}
				// 初始化坐标
				this.initXY=[event.touches[0].pageX,event.touches[0].pageY];
			},
			handleTouchMove(index,event){
				// 多点触控 不触发
				if(event.touches.length>1){
					return 
				}
				// 移动位置
				let moveX=event.touches[0].pageX-this.initXY[0];
				let moveY=event.touches[0].pageY-this.initXY[1];
				// 如果滑动位置距离小
				if(Math.abs(moveX)<5){
					return
				}
				// 竖向滑动不触发
				if(Math.abs(moveY)>Math.abs(moveX)){
					return
				}
				// 左滑
				if(moveX<0){
					this.theIndex=index;
				}else{
					// 右滑
					this.oldIndex=this.theIndex;
					this.theIndex=null;
				}
			},
			handleTouchEnd(index,event){
				
			},
			handleSelectAll(){
				let arr=[];
				this.selectAll=!this.selectAll;
				this.goodsList.forEach((item)=>{
					item.selected=this.selectAll;
					arr.push(item)
				})
				this.selectList=this.selectAll?arr:[]
			},
			handleSingleDelete(item){
				uni.getStorage({
					key:"goodsList",
					success:(res)=>{
						res.data.splice(res.data.indexOf(item),1);
						uni.setStorageSync("goodsList",res.data);
					}
				})
				this.goodsList.splice(this.goodsList.indexOf(item),1);
				this.selectList.splice(this.selectList.indexOf(item),1);
				this.theIndex=null;
			},
			handleMulDelete(){
				while(this.selectList.length>0){
					this.handleSingleDelete(this.selectList[0]);
				}
				this.selectList=[];
				this.selectAll=false;
			},
			handleConfirm(){
				if(this.selectList.length<1){
					uni.showToast({
						title:"请选择结算的商品",
						icon:'none'
					})
					return
				}
				//本地存储
				uni.setStorage({
					key:"confirmList",
					data:this.selectList,
					success:()=>{
						uni.navigateTo({
							url:"../../order/order"
						})
					}
				})
				console.log(this.selectList)
			},
			add(item){
				item.number++
			},
			sub(item){
				if(item.number<=1){
					return
				}
				item.number--
			}
		}
	}
</script>

<style lang="scss">
	.container {
		display: flex;
		align-items: center;

		.checkbox {
			width: 35upx;
			height: 35upx;
			border-radius: 100%;
			border: solid 2upx #f06c7a;
			display: flex;
			justify-content: center;
			align-items: center;

			.on {
				width: 25upx;
				height: 25upx;
				border-radius: 100%;
				background-color: #f06c7a;
			}
		}

		.text {
			font-size: 28upx;
			margin-left: 10upx;
		}
	}


	.goods-list {
		width: 100%;
		padding: 20upx 0 120upx 0;

		.empty {
			width: 100%;
			height: 60upx;
			display: flex;
			justify-content: center;
			align-items: center;
			font-size: 32upx;
			color: #a7a7a7;
		}

		.row {
			width: calc(92%);
			height: calc(22vw + 40upx);
			margin: 20upx auto;

			border-radius: 15upx;
			box-shadow: 0upx 5upx 20upx rgba(0, 0, 0, 0.1);
			display: flex;
			align-items: center;
			position: relative;
			overflow: hidden;
			z-index: 4;
			border: 0;

			.menu {
				.icon {
					color: #fff;
					font-size: 60upx;
				}

				position: absolute;
				width: 30%;
				height: 100%;
				right: 0;
				display: flex;
				justify-content: center;
				align-items: center;
				background-color: red;
				color: #fff;
				z-index: 2;
			}

			.production {
				@keyframes showMenu {
					0% {
						transform: translateX(0);
					}

					100% {
						transform: translateX(-30%);
					}
				}

				@keyframes closeMenu {
					0% {
						transform: translateX(-30%);
					}

					100% {
						transform: translateX(0);
					}
				}

				&.open {
					animation: showMenu 0.25s linear both;
				}

				&.close {
					animation: closeMenu 0.15s linear both;
				}

				background-color: #fff;

				.container {
					padding-left: 20upx;
					flex-shrink: 0;
					height: 22vw;
					margin-right: 20upx;
				}

				position: absolute;
				width: 100%;
				padding: 0 0;
				height: 100%;
				z-index: 3;
				display: flex;
				align-items: center;

				.goods-info {
					width: 100%;
					display: flex;
					padding-right: 20upx;

					.img {
						width: 22vw;
						height: 22vw;
						border-radius: 10upx;
						overflow: hidden;
						flex-shrink: 0;
						margin-right: 10upx;

						image {
							width: 22vw;
							height: 22vw;
						}
					}

					.info {
						width: 100%;
						height: 22vw;
						overflow: hidden;
						display: flex;
						flex-wrap: wrap;
						position: relative;

						.title {
							width: 100%;
							font-size: 28upx;
							display: -webkit-box;
							-webkit-box-orient: vertical;
							-webkit-line-clamp: 2;
							// text-align: justify;
							overflow: hidden;
						}

						.spec {
							font-size: 20upx;
							background-color: #f3f3f3;
							color: #a7a7a7;
							height: 30upx;
							display: flex;
							align-items: center;
							padding: 0 10upx;
							border-radius: 15upx;
							margin-bottom: 20vw;
						}

						.price-number {
							position: absolute;
							width: 100%;
							bottom: 0upx;
							display: flex;
							justify-content: space-between;
							align-items: flex-end;
							font-size: 28upx;
							height: 60upx;

							.price {}

						}
					}
				}
			}
		}
	}

	.footer {
		width: 92%;
		padding: 0 4%;
		background-color: #fbfbfb;
		height: 100upx;
		display: flex;
		justify-content: space-between;
		align-items: center;
		font-size: 28upx;
		position: fixed;
		bottom: 0upx;
		z-index: 5;

		.delBtn {
			border: solid 1upx #f06c7a;
			color: #f06c7a;
			padding: 0 30upx;
			height: 50upx;
			border-radius: 30upx;
			display: flex;
			justify-content: center;
			align-items: center;
		}

		.settlement {
			width: 60%;
			display: flex;
			justify-content: flex-end;
			align-items: center;

			.sum {
				width: 50%;
				font-size: 28upx;
				margin-right: 10upx;
				display: flex;
				justify-content: flex-end;

				.money {
					font-weight: 600;
				}
			}

			.btn {
				padding: 0 30upx;
				height: 50upx;
				background-color: #f06c7a;
				color: #fff;
				display: flex;
				justify-content: center;
				align-items: center;

				border-radius: 30upx;
			}
		}
	}
</style>

