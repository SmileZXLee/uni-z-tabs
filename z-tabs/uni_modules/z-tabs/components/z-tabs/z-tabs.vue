<template name="z-tabs">
	<view class="z-tabs-conatiner">
		<slot name="left" />
		<view class="z-tabs-scroll-view-conatiner">
			<scroll-view ref="z-tabs-scroll-view" class="z-tabs-scroll-view" :scroll-x="shouldScroll" :scroll-left="scrollLeft" :show-scrollbar="false" :scroll-with-animation="isFirstLoaded" @scroll="scroll">
				<view class="z-tabs-list-container" :style="[tabsListStyle]">
					<view class="z-tabs-list" :style="[tabsListStyle]">
						<view :ref="`z-tabs-item-${index}`" :id="`z-tabs-item-${index}`" class="z-tabs-item" :style="[tabsStyle]" v-for="(item,index) in list" :key="index" @click="tabsClick(index,item)">
							<view class="z-tabs-item-title-container">
								<text class="z-tabs-item-title" :style="[{color:currentIndex===index?activeColor:inactiveColor},currentIndex===index?activeStyle:inactiveStyle]">{{item[nameKey]||item}}</text>
							</view>
						</view>
					</view>
					<view class="z-tabs-bottom">
						<view 
						<!-- #ifndef APP-NVUE -->
						v-if="bottomDotX > 0" 
						<!-- #endif -->
						ref="z-tabs-bottom-dot" class="z-tabs-bottom-dot"
						<!-- #ifdef APP-VUE || MP-WEIXIN || MP-QQ || H5 -->
						:change:prop="tabsWxs.propObserver" :prop="wxsPropType"
						:style="[{background:activeColor}]"
						<!-- #endif -->
						<!-- #ifndef APP-VUE || MP-WEIXIN || MP-QQ || H5 || APP-NVUE -->
						:style="[{transform:`translateX(${bottomDotX}px)`,transition:dotTransition,background:activeColor}]"
						<!-- #endif -->
						<!-- #ifdef APP-NVUE -->
						:style="[{background:activeColor}]"
						<!-- #endif -->
						/>
					</view>	
				</view>
			</scroll-view>
		</view>
		<slot name="right" />
	</view>
</template>

<!-- #ifdef APP-VUE || MP-WEIXIN || MP-QQ || H5 -->
<script src="./wxs/z-tabs-wxs.wxs" module="tabsWxs" lang="wxs"></script>
<!-- #endif -->
<script>
	// #ifdef APP-NVUE
	const weexDom = weex.requireModule('dom');
	const weexAnimation = weex.requireModule('animation');
	// #endif
	/**
	 * z-tabs 标签
	 * @description 一个简单轻量的tabs标签，全平台兼容，支持nvue、vue3
	 * @tutorial 
	 * @property {Array} list 数据源数组
	 * @property {Number|String} current 当前选中的index
	 * @property {Number|String} current 当前选中的index
	 * @event {Function(index)} change 标签改变时触发 index: 点击了第几个tab，索引从0开始
	 * @event {Function(index)} click 点击标签时触发 index: 点击了第几个tab，索引从0开始
	 * @example <u-tabs :list="list" :is-scroll="false" :current="current" @change="change"></u-tabs>
	 */
	export default {
		name: 'z-tabs',
		data() {
			return {
				currentIndex: 0,
				bottomDotX: 0,
				
				scrollLeft: 0,
				wxsPropType: '',
				tabsWidth: 0,
				tabsContainerWidth: 0,
				itemNodeInfos: [],
				isFirstLoaded: false,
				currentScrollLeft: 0
			};
		},
		props: {
			list: {
				type: Array,
				default: function() {
					return [];
				}
			},
			current: {
				type: Number|String,
				default: 0
			},
			scrollCount: {
				type: Number,
				default: 5
			},
			tabWidth: {
				type: Number,
				default: 0
			},
			nameKey: {
				type: String,
				default: 'name'
			},
			valueKey: {
				type: String,
				default: 'value'
			},
			activeColor: {
				type: String,
				default: '#007AFF'
			},
			inactiveColor: {
				type: String,
				default: '#888888'
			},
			activeStyle: {
				type: Object,
				default: function() {
					return {};
				}
			},
			inactiveStyle: {
				type: Object,
				default: function() {
					return {};
				}
			}
		},
		mounted() {
			this.$nextTick(()=>{
				this._getNodeClientRect(`.z-tabs-scroll-view`).then(res=>{
					if(res && res.length){
						this.tabsWidth = res[0].width;
					}
				})
			})
		},
		watch: {
			current: {
				handler(newVal) {
					if(newVal > 0){
						this.currentIndex = newVal;
						this._updateDotPosition(this.currentIndex);
					}
				},
				immediate: true
			},
			list: {
				handler(newVal) {
					this.$nextTick(async ()=>{
						if(newVal.length){
							this.tabsContainerWidth = 0;
							this.itemNodeInfos = [];
							let delayTime = 0;
							// #ifdef APP-VUE
							delayTime = 200;
							// #endif
							setTimeout(async()=>{
								for(let i = 0;i < newVal.length;i++){
									const nodeRes = await this._getNodeClientRect(`#z-tabs-item-${i}`,true);
									if(nodeRes && nodeRes.length){
										const node = nodeRes[0];
										this.itemNodeInfos.push(node);
										this.tabsContainerWidth += node.width;
									}
								}
								this._updateDotPosition(this.currentIndex);
							},delayTime)
						}
					})
				},
				immediate: true
			},
			bottomDotX(newVal) {
				// #ifdef APP-VUE || MP-WEIXIN || MP-QQ || H5
				this.wxsPropType = {transformValue:newVal,transition:this.dotTransition};
				// #endif
				// #ifdef APP-NVUE
				weexAnimation.transition(this.$refs['z-tabs-bottom-dot'], {
					styles: {
						transform: `translateX(${newVal}px)`,
					},
					duration: this.isFirstLoaded ? 200 : 0
				})
				// #endif
			}
		},
		computed: {
			shouldScroll(){
				return this.list.length > this.scrollCount;
			},
			tabsStyle(){
				const stl = this.shouldScroll ? {'flex-shrink': 0} : {'flex': 1};
				if(this.tabWidth > 0){
					stl['width'] = this.tabWidth + 'rpx';
				}else{
					delete stl.width;
				} 
				return stl;
			},
			tabsListStyle(){
				return this.shouldScroll ? {} : {'flex':1};
			},
			dotTransition(){
				return this.isFirstLoaded?'transform .2s linear':'none';
			}
		},
		methods: {
			//点击了tabs
			tabsClick(index,item) {
				if (this.currentIndex != index) {
					this.$emit('change', index, item);
					this.currentIndex = index;
					this._updateDotPosition(index);
				}
			},
			scroll(e){
				this.currentScrollLeft = e.detail.scrollLeft;
			},
			//更新底部dot位置
			_updateDotPosition(index){
				if(index >= this.itemNodeInfos.length) return;
				this.$nextTick(async ()=>{
					let node = this.itemNodeInfos[index];
					let offset = 0;
					let tabsContainerWidth = this.tabsContainerWidth;
					if (JSON.stringify(this.activeStyle) !== '{}') {
						const nodeRes = await this._getNodeClientRect(`#z-tabs-item-${index}`,true);
						if (nodeRes && nodeRes.length){
							node = nodeRes[0];
							offset = this.currentScrollLeft;
							tabsContainerWidth = 0;
							for(let i = 0;i < this.itemNodeInfos.length;i++){
								let oldNode = this.itemNodeInfos[i];
								tabsContainerWidth += i === index ? node.width : oldNode.width;
							}
						}
					}
					this.bottomDotX = node.left + node.width / 2 - uni.upx2px(34) / 2 + offset;
					if(this.tabsWidth){
						setTimeout(()=>{
							let scrollLeft = this.bottomDotX - this.tabsWidth / 2 + uni.upx2px(34) / 2;
							scrollLeft = Math.max(0,scrollLeft);
							if(tabsContainerWidth){
								scrollLeft = Math.min(scrollLeft,tabsContainerWidth - this.tabsWidth + 10);
							}
							this.scrollLeft = scrollLeft;
							this.$nextTick(()=>{
								this.isFirstLoaded = true;
							})
						},200)
					}
				})
			},
			//获取节点信息
			_getNodeClientRect(select, withRefArr = false) {
				// #ifdef APP-NVUE
				select = select.replace('.', '').replace('#', '');
				const ref = withRefArr ? this.$refs[select][0] : this.$refs[select];
				return new Promise((resolve, reject) => {
					if (ref) {
						weexDom.getComponentRect(ref, option => {
							if (option && option.result) {
								resolve([option.size]);
							} else resolve(false);
						})
					} else resolve(false);
				});
				return;
				// #endif
				const res = uni.createSelectorQuery().in(this);
				res.select(select).boundingClientRect();
				return new Promise((resolve, reject) => {
					res.exec(data => {
						resolve((data && data != '' && data != undefined && data.length) ? data : false);
					});
				});
			}
		}
	}
</script>

<style scoped>
	.z-tabs-conatiner{
		/* #ifndef APP-NVUE */
		overflow: hidden;
		display: flex;
		/* #endif */
		width: 750rpx;
		height: 80rpx;
		padding: 10rpx 0px;
		flex-direction: row;
	}
	
	.z-tabs-scroll-view-conatiner{
		flex: 1;
		position: relative;
		/* #ifndef APP-NVUE */
		display: flex;
		/* #endif */
		flex-direction: row;
	}
	
	/* #ifndef APP-NVUE */
	.z-tabs-scroll-view ::-webkit-scrollbar {
		display: none;
		-webkit-appearance: none;
		width: 0 !important;
		height: 0 !important;
		background: transparent;
	}
	/* #endif */
	
	.z-tabs-scroll-view{
		flex-direction: row;
		position: absolute;
		left: 0;
		top: 0;
		right: 0;
		bottom: 0;
		/* #ifndef APP-NVUE */
		width: 100%;
		height: 100%;
		/* #endif */
		flex: 1;
	}
	
	.z-tabs-list-container{
		padding-bottom: 18rpx;
		position: relative;
	}
	
	.z-tabs-list,.z-tabs-list-container{
		/* #ifndef APP-NVUE */
		display: flex;
		/* #endif */
		flex-direction: row;
	}
	
	.z-tabs-item{
		/* #ifndef APP-NVUE */
		display: flex;
		/* #endif */
		flex-direction: row;
		justify-content: center;
		align-items: center;
		padding: 0px 20rpx;
	}
	
	.z-tabs-item-title{
		font-size: 30rpx;
	}
	
	.z-tabs-bottom{
		position: absolute;
		bottom: 0;
		width: 100%;
	}
	
	.z-tabs-bottom-dot{
		height: 8rpx;
		width: 36rpx;
		border-radius: 100px;
	}
</style>
