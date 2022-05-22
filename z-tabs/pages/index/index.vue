<template>
	<view class="content">
		<view class="group">
			<text class="group-title">
				基本使用
			</text>	
			<z-tabs :list="list"></z-tabs>
		</view>
		
		<view class="group">
			<text class="group-title">
				自定义高亮颜色
			</text>	
			<z-tabs :list="list" active-color="orange"></z-tabs>
		</view>
		<view class="group">
			<text class="group-title">
				自定义高亮style
			</text>	
			<z-tabs :list="list" :active-style="{'font-size':'40rpx','font-weight':'bold'}"></z-tabs>
		</view>
		<view class="group">
			<text class="group-title">
				自定义初始选中tab
			</text>	
			<z-tabs :list="list" :current="5"></z-tabs>
		</view>
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
				list: []
			}
		},
		onLoad() {
			for(let i = 0;i < 10;i++){
				this.list.push('tab' + (i + 1));
			}
		},
		methods: {
			gotoNvue(){
				uni.navigateTo({
					url: '/pages/index-nvue/index-nvue'
				})
			}
		}
	}
</script>

<style>
	.group{
		margin-top: 30rpx;
	}
	.group-title{
		padding-left: 20rpx;
		font-size: 26rpx;
		color: #aaaaaa;
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
