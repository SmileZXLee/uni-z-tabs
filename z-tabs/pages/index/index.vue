<template>
	<view class="content">
		<!-- 基本使用 -->
		<view class="group">
			<text class="group-title">
				基本使用
			</text>	
			<z-tabs :list="list"></z-tabs>
		</view>
		
		<!-- 自定义高亮颜色 -->
		<view class="group">
			<text class="group-title">
				自定义高亮颜色
			</text>	
			<z-tabs :list="list" active-color="orange"></z-tabs>
		</view>
		
		<!-- 自定义高亮style -->
		<view class="group">
			<text class="group-title">
				自定义高亮style
			</text>	
			<z-tabs :list="list" :active-style="{'font-size':'40rpx','font-weight':'bold'}"></z-tabs>
		</view>
		
		<!-- 自定义初始选中tab -->
		<view class="group">
			<text class="group-title">
				自定义初始选中tab
			</text>	
			<z-tabs :list="list" :current="5"></z-tabs>
		</view>
		
		<!-- 显示徽标数 -->
		<view class="group">
			<text class="group-title">
				显示徽标数
			</text>	
			<z-tabs :list="badgeList"></z-tabs>
		</view>
		
		<!-- 自定义右侧插槽 -->
		<view class="group">
			<text class="group-title">
				自定义右侧插槽
			</text>	
			<z-tabs :list="list">
				<template v-slot:right>
					<view style="margin: 0 10rpx;">🏡</view>
				</template>
			</z-tabs>
		</view>
		
		<!-- 与swiper联动 -->
		<view class="group">
			<text class="group-title">
				与swiper联动
			</text>	
			<!-- 可通过设置bar-animate-mode="worm"开启毛毛虫模式-->
			<z-tabs ref="tabs" :current="swiperCurrent" :list="swiperList" @change="tabsChange"></z-tabs>
			<swiper class="swiper" :current="swiperCurrent" @transition="swiperTransition" @animationfinish="swiperAnimationfinish">
			    <swiper-item v-for="(item, index) in swiperList" :key="index">
					<view class="swiper-item-view" :style="{backgroundColor: item.color}">
						{{item.name}}
					</view>
			    </swiper-item>
			</swiper>
		</view>
		
		<!-- 二次点击 -->
		<view class="group">
			<text class="group-title">
				二次点击
			</text>	
			<z-tabs :list="list" enableSecondClick @secondClick="secondClick"></z-tabs>
		</view>
		
		<!-- #ifdef APP-PLUS -->
		<view class="to-nvue-btn" @click="gotoNvue">
			查看nvue-demo
		</view>
		<!-- #endif -->
		<!-- #ifndef APP-PLUS -->
		<view class="notice">
			—— 将demo运行至APP平台可查看nvue中的运行效果 ——
		</view>
		<!-- #endif -->
	</view>
</template>

<script>
	export default {
		data() {
			return {
				list: [],
				badgeList: [],
				swiperList: [],
				swiperCurrent: 0
			}
		},
		onLoad() {
			//普通情况数组
			const list = [];
			for(let i = 0;i < 10;i++) {
				list.push('tab' + (i + 1));
			}
			this.list = list;
		
			//自定义徽标数数组
			const badgeList = [];
			for(let i = 0;i < 10;i++) {
				if(i !== 2){
					badgeList.push({
						name: 'tab' + (i + 1),
					});
				}else {
					badgeList.push({
						name: 'tab' + (i + 1),
						badge: {
							count: 6
						}
					});
				}
			}
			this.badgeList = badgeList;
			
			//与swiper联动数组
			const swiperList = [];
			for(let i = 0;i < 4;i++) {
				swiperList.push({
					name: 'tab' + (i + 1),
					color: this._getRandomColor()
				});
			}
			this.swiperList = swiperList;
		},
		methods: {
			gotoNvue(){
				uni.navigateTo({
					url: '/pages/index-nvue/index-nvue'
				})
			},
			
			//tabs通知swiper切换
			tabsChange(index) {
				this.swiperCurrent = index;
			},
			//swiper滑动中
			swiperTransition(e) {
				this.$refs.tabs.setDx(e.detail.dx);
			},
			//swiper滑动结束
			swiperAnimationfinish(e) {
				this.swiperCurrent = e.detail.current;
				this.$refs.tabs.unlockDx();
			},
			
			// 生成随机颜色
			_getRandomColor() {
				const rgb = [];
				for (let i = 0; i < 3; ++i) {
					let color = Math.floor(Math.random() * 256).toString(16)
					color = color.length == 1 ? '0' + color : color
					rgb.push(color)
				}
				return '#' + rgb.join('');
			},
			
			//二次点击
			secondClick(index,item){
				uni.showToast({
					title:`tab${index+1},二次点击确认`,
					icon:'none'
				})
			}
		}
	}
</script>

<style>
	.content{
		padding-bottom: 20rpx;
	}
	.group{
		margin-top: 30rpx;
	}
	.group-title{
		padding-left: 20rpx;
		font-size: 26rpx;
		color: #aaaaaa;
	}
	.swiper{
		height: 300rpx;
	}
	.swiper-item-view{
		background-color: #007AFF;
		height: 300rpx;
		display: flex;
		justify-content: center;
		align-items: center;
		color: white;
		font-size: 50rpx;
	}
	.to-nvue-btn{
		background-color: #007AFF;
		height: 80rpx;
		color: white;
		text-align: center;
		line-height: 80rpx;
		width: calc(100% - 40rpx);
		margin-left: 20rpx;
		margin-top: 20rpx;
		border-radius: 10rpx;
	}
	.notice{
		text-align: center;
		color: #aaaaaa;
		font-size: 24rpx;
		margin-top: 30rpx;
	}
</style>
