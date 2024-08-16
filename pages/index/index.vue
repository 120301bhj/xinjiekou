<template>
	<view class="container">
		<view class="swiper-box">
			<view v-if="!examType"
				style="text-align: center;padding:30rpx 0 0rpx 0;display: flex;justify-content: center;align-items: center;">
				<image style="width: 50rpx;height: 50rpx;margin-right: 5rpx;position: relative;bottom: 3rpx;"
					:src="'http://wfqn.xllcedupro.top/'+'upload_pic_闹钟.png'" mode=""></image>
				<u-count-down ref="countDown" @finish="changeFinish" :time="45*60*1000" format="mm:ss"></u-count-down>
			</view>
			<swiper class="swiper" :duration="duration" :current="swiperIndex" :disable-programmatic-animation="true"
				@change="onChange" @animationfinish="onAnimationfinish">
				<swiper-item v-for="(item,index) in swiperList" :key="index">
					<scroll-view scroll-y="true" class="swiper-scroll">
						<view class="swiper-item" @touchstart="clearAutoNextTopicTime">
							<view class="topic-content">
								<view>
									<text style="padding-right: 20rpx;">
										<text class="type-label">
											{{item.type | topicType}}
										</text>
									</text>
									<text @click.stop="btnBackMusicType(item.audio)">{{item.title}}<text
											v-if='!backMusicType'
											style='font-size:30rpx;position: relative;top:3rpx;background-color:#3598FD;border-radius: 50%;padding: 5rpx;margin-left: 10rpx;'
											class="cuIcon-playfill xl. text-white"></text>
										<text v-if='backMusicType'
											style='font-size:30rpx;position: relative;top:3rpx;background-color:#3598FD;border-radius: 50%;padding: 5rpx;margin-left: 10rpx;'
											class="cuIcon-stop lg. text-white"></text></text>
								</view>
								<!-- 图片以及音频 -->
								<view v-if="item.image!='' || item.video && item.video!=''"
									style="height: 300rpx;display: flex;flex-direction: column;align-items: center;margin-top: 20rpx;">
									<image v-if="item.image!=''" style="width: 90%;height: 300rpx;"
										:src="'http://wfqn.xllcedupro.top/'+item.image" mode=""></image>
									<video :show-progress='false' :show-fullscreen-btn='false' :show-play-btn='false'
										:show-center-play-btn='false' autoplay loop v-if="item.video && item.video!=''"
										style="width: 90%;height: 300rpx;"
										:src="'http://wfqn.xllcedupro.top/'+item.video"></video>
								</view>
								<view v-if="item.type!='2'" class="answer-sheet">
									<view class="item h-flex-x" v-for="(sheetItem,sheetIndex) in item.answerSheet"
										:key="sheetIndex" @tap="onAnswerSheet(sheetIndex)">
										<view class="icon h-flex-x h-flex-center" :class="{
												'success':(item.answerResult == sheetItem.value) && (item.answer == sheetItem.value),
												'error':(item.answerResult == sheetItem.value) && (item.answer != sheetItem.value)
											}">
											<block v-if="item.answerResult == sheetItem.value">
												<!-- 判断 答题之后展示icon图标不同 -->
												<uni-icons type="checkmarkempty" size="12" color="#fff"
													v-if="item.answer == sheetItem.value"></uni-icons>
												<uni-icons type="closeempty" size="12" color="#fff" v-else></uni-icons>
											</block>
											<text v-else>{{sheetItem.value}}</text>
										</view>
										<view>{{sheetItem.name}}</view>
									</view>
								</view>
								<!-- 	:class="{
												'success':(item.answerResult == sheetItem.value) && (item.answer == sheetItem.value),
												'error':(item.answerResult == sheetItem.value) && (item.answer != sheetItem.value)
											}"> -->
								<view v-if="item.type=='2'" class="answer-sheet">
									<view class="item h-flex-x" v-for="(sheetItem,sheetIndex) in item.answerSheet"
										:key="sheetIndex" @tap="onAnswerSheets(sheetIndex)">
										<view class="icon h-flex-x h-flex-center" :class="{'success': !item.isShow && item.answerResult && 
										(item.answerResult==sheetItem.value || item.answerResult.indexOf(sheetItem.value) !=-1),
										'success': item.isShow && item.answer.indexOf(sheetItem.value) !=-1,
                                        'error': item.isShow && item.answer.indexOf(sheetItem.value) ==-1 && item.answerResult.split(',').sort().toString()!=item.answer.split(',').sort().toString()
											}">
											<block
												v-if="!item.isShow && item.answerResult && (item.answerResult==sheetItem.value || item.answerResult.indexOf(sheetItem.value) !=-1)">
												<!-- 确认答案之前 展示一样的图标 -->
												<uni-icons type="checkmarkempty" size="12" color="#fff"></uni-icons>
											</block>
											<block
												v-else-if="item.isShow  && item.answer.indexOf(sheetItem.value) !=-1">
												<!-- 判断 确认答案之后 展示对的图标-->
												<uni-icons type="checkmarkempty" size="12" color="#fff"></uni-icons>
											</block>
											<block
												v-else-if="item.isShow && item.answerResult.split(',').sort().toString()!=item.answer.split(',').sort().toString() && item.answer.indexOf(sheetItem.value) ==-1">
												<!-- 判断 确认答案之后 展示错的图标-->
												<uni-icons type="closeempty" size="12" color="#fff"></uni-icons>
											</block>
											<text v-else>{{sheetItem.value}}</text>
										</view>
										<view>{{sheetItem.name}}</view>
									</view>
									<view style='margin: 20rpx 60rpx;'>
										<u-button @click="btnCheckbox(sheetIndex)" shape="circle"
											color="rgb(53, 152, 253)">确认答案
										</u-button>
									</view>
								</view>
								<!--  -->
								<view class="answer-result" v-if="item.type=='2' && item.isShow">
									<text>答案</text>
									<text
										style="color: #0089ff;font-weight: bold;padding: 0 20rpx;">{{item.answer}}</text>
									<text style="padding-right: 20rpx;">您选择</text>
									<text style="color: #0089ff;font-weight: bold;"
										v-if="isShow && item.answerResult == item.answer">{{item.answerResult}}</text>
									<text style="color: #f84d27;font-weight: bold;" v-else>{{item.answerResult}}</text>
								</view>
								<view class="answer-doubt" v-if="item.type=='2' && item.isShow">
									<text style="font-weight: bold;">题目解析：</text>
									<text>{{item.answerDoubt}}</text>
								</view>

								<view class="answer-result" v-if="item.type!='2' && item.answerResult">
									<text>答案</text>
									<text
										style="color: #0089ff;font-weight: bold;padding: 0 20rpx;">{{item.answer}}</text>
									<text style="padding-right: 20rpx;">您选择</text>
									<text style="color: #0089ff;font-weight: bold;"
										v-if="isShow && item.answerResult == item.answer">{{item.answerResult}}</text>
									<text style="color: #f84d27;font-weight: bold;" v-else>{{item.answerResult}}</text>
								</view>
								<view class="answer-doubt" v-if="item.type!='2' && item.answerResult">
									<text style="font-weight: bold;">题目解析：</text>
									<text>{{item.answerDoubt}}</text>
								</view>
							</view>
						</view>
					</scroll-view>
				</swiper-item>
			</swiper>
		</view>
		<view style='padding-top: 300rpx;'>
			<u-empty v-if="dataList.length<=0" mode="data" icon="http://cdn.uviewui.com/uview/empty/car.png"></u-empty>
		</view>

		<view class="panel-bottom">
			<view class="shade" v-if="showPopup" @tap="showPopup = !showPopup"></view>
			<view class="content">
				<view class="action-bar h-flex-x">
					<view v-if="!collectType" @tap="addFavorite">
						<view class="h-flex-x" v-if="isFavorite">
							<uni-icons type="star-filled" size="18" color="#f2ce1b"></uni-icons>
							<text style="padding-left: 10rpx;">已收藏</text>
						</view>
						<view class="h-flex-x" v-else>
							<uni-icons type="star" size="18" color="#666"></uni-icons>
							<text style="padding-left: 10rpx;">收藏</text>
						</view>
					</view>
					<view class="h-flex-item-grow h-flex-x h-flex-center" @tap="showPopup = !showPopup">
						<view>
							<uni-icons type="checkbox-filled" size="18" color="#0089ff"></uni-icons>
						</view>
						<view class="success-num" style="padding-left: 10rpx;margin-right: 30rpx;">
							{{countResult.success}}
						</view>
						<view>
							<uni-icons type="clear" size="18" color="#f84d27"></uni-icons>
						</view>
						<view class="error-num" style="padding-left: 10rpx;">{{countResult.error}}</view>

					</view>

					<view @click="btnRemove" v-if="errorType" style="margin-right: 20rpx;background-color:#0089ff;color:#FFFFFF;
						padding: 7rpx 17rpx;font-size: 26rpx;border-radius: 22rpx;">清空错题</view>
					<view @click="btnExam" style="margin-right: 20rpx;background-color:#0089ff;color:#FFFFFF;
							padding: 10rpx 40rpx;font-size: 26rpx;border-radius: 22rpx;">交卷</view>

					<view class="h-flex-x" @tap="showPopup = !showPopup">
						<uni-icons type="bars" size="18" color="#666"></uni-icons>
						<text style="padding-left: 10rpx;font-weight: bold;">{{topicIndex+1}}</text>
						<text style="color: #999;padding: 0 5rpx;">/</text>
						<text style="color: #999;">{{dataList.length}}</text>
					</view>
				</view>
				<scroll-view class="topic-list" v-if="showPopup" style="height: 700rpx;" scroll-y="">
					<view class="list-box h-flex-x h-flex-wrap h-flex-6">
						<view class="list-item" v-for="(item,index) in dataList" :key="index" @tap="pickerTopic(index)">
							<view class="h-flex-x h-flex-center" :class="{
									'active':index == topicIndex,
									'success':item.answerResult && (item.answer == item.answerResult),
									'error':item.answerResult && (item.answer != item.answerResult)
								}">{{index+1}}</view>
						</view>
					</view>
				</scroll-view>
			</view>
		</view>
		<u-modal :showCancelButton='true' @cancel="cancelExam" @confirm='confirmExam' confirmText='继续答题'
			cancelText='现在交卷' :show="modelShow" :title="title">
			<view class="slot-content" style="width: 100%;height: 300rpx;">
				<view style="width: 70%;margin: 20rpx auto;text-align: center;height: 300rpx;" class="">
					<view style="margin-bottom: 60rpx;font-size: 38rpx;font-weight: bold;">提示
					</view>
					<u-line-progress height="15" :percentage="percentage" activeColor="#0089ff"></u-line-progress>
					<view style="margin-top: 60rpx;font-size: 25rpx;">
						您还有{{topicPage}}题未做
					</view>
				</view>
			</view>
		</u-modal>

	</view>
</template>

<script>
	const innerAudioContext = uni.createInnerAudioContext();
	import uniIcons from '@/components/uni-icons/uni-icons';
	// const dataApiOne = require('../../utils/sort_one.json');
	// import dataApiOne from '../../utils/sort_one.js'
	import dataApiFour from '@/static/list.js'
	// const dataApiFour = require('../../utils/sort_four.json');
	// import {
	// 	topicIndexs,
	// 	topicError,
	// 	topicCollect,
	// 	addTopicError,
	// 	topicCollectGet,
	// 	topicEmpty,
	// 	examineStart,
	// 	examineStop,
	// 	topicSkill
	// } from '@/api/home.js'
	export default {
		components: {
			uniIcons,
		},
		data() {
			return {
				answerResultString: [],
				dataList: [], // 数据源
				swiperList: [], // 轮播图数据列表
				swiperIndex: 0, // 轮播图当前位置
				isChange: false, // 是否切换
				topicIndex: 0, // 题目位置
				duration: 200, // 动画过渡时长
				showPopup: false, //弹窗是否显示
				backMusicType: true,
				count: 0,
				page: 2,
				calssId: '',
				swiperTypeGet: true,
				pageIndex: null,
				errorType: false,
				collectType: false,
				examType: false,
				modelShow: false,
				percentage: 0,
				topicPage: 0,
				examId: null,
				sort_id: null,
			}
		},
		computed: {
			// 结果统计
			countResult() {
				let [success, error] = [0, 0];
				this.dataList.forEach((item) => {
					if (item.answerResult) {
						if (item.answerResult == item.answer) {
							success++;
						} else {
							error++;
						}
					}
				})
				return {
					success,
					error
				}
			},
			// 是否收藏
			isFavorite() {
				let item = this.dataList[this.topicIndex];
				if (item && item.isCollect) {
					return true;
				}
				return false;
			}
		},
		filters: {
			topicType(type) {
				if (type == 0) {
					return '选择题';
				}
				if (type == 1) {
					return '判断题';
				}
				if (type == 2) {
					return '多选题';
				}
				return ''
			}
		},
		onLoad(options) {
			// uni.setNavigationBarTitle({
			// 	title: this.language.index.craftNavbarText
			// class_id为分类练习
			if (options.class_id) {
				this.calssId = options.class_id
			}
			// error为错题练习
			if (options.error) {
				this.errorType = true
			}
			if (options.collect) {
				this.collectType = true
			}
			if (options.exam) {
				this.examType = true
			}
			// this.sort_id = options.sort_id
			this.getList()
		},
		onUnload() {
			innerAudioContext.stop()
			console.log('返回上一页');
		},
		methods: {
			getList() {
				// 分类练习 or 顺序练题
				uni.showLoading({
					title: '加载中...'
				})
				let res = {}
				res.data = dataApiFour.craftFourList()
				// topicSkill({
				// 	type: uni.getStorageSync('subject'),
				// }).then(Response => {
				// 	res.data = Response.data.filter(item => {
				// 		return item.sort_id == this.sort_id
				// 	})
				uni.hideLoading()
				console.log(res.data, '顺序以及分类接口返回数据');
				this.count = res.data.length
				if (res.data.length > 0) {
					for (let i = 0; i < res.data.length; i++) {
						let arr = []
						let answer = []
						let options = res.data[i].options.map(item => {
							return {
								"value": item.tag,
								"name": item.content
							}
						})
						if (res.data[i].type.toString() != '2') {
							answer = res.data[i].options.filter(item => {
								return item.isCorrect == 1
							})
						} else {
							for (let var1 in res.data[i].options) {
								if (res.data[i].options[var1].isCorrect == 1) {
									arr.push(res.data[i].options[var1].tag)
								}
							}
						}
						let obj = [{
							'type': res.data[i].type.toString(),
							'title': res.data[i].title,
							'topicId': res.data[i].topicId,
							'audio': res.data[i].audio,
							'image': res.data[i].image,
							'answerDoubt': res.data[i].analysis,
							"answerSheet": options,
							"answer": res.data[i].type.toString() == '2' ? arr.join(',') : answer[0]
								.tag,
							"video": res.data[i].video,
							"errAudio": res.data[0].error_audio,
							"isCollect": res.data[0].isCollect == 1 ? true : false,
							"isShow": false,
						}]
						this.dataList = this.dataList.concat(obj)
					}
					if (this.swiperTypeGet) {
						this.renderSwiper(0);
						innerAudioContext.src = 'http://wfqn.xllcedupro.top/' + encodeURIComponent(this.dataList[0].audio);
						innerAudioContext.autoplay = true;
						innerAudioContext.loop = true
						innerAudioContext.obeyMuteSwitch = false;
						innerAudioContext.play();
					} else {
						// console.log(this.swiperIndex,'this.swiperIndex');
						if (this.pageIndex != 0) {
							// console.log(this.pageIndex-1,'this.pageIndex-1');
							this.renderSwiper(this.pageIndex - 1);
						}
					}
				}
				// })
			},
			btnBackMusicType(value) {
				if (!this.backMusicType) {
					innerAudioContext.src = 'http://wfqn.xllcedupro.top/' + encodeURIComponent(value);
					innerAudioContext.autoplay = true;
					innerAudioContext.loop = true
					innerAudioContext.obeyMuteSwitch = false;
					innerAudioContext.play();
					this.backMusicType = true
				} else if (this.backMusicType) {
					innerAudioContext.stop()
					this.backMusicType = false
				}
			},
			// 渲染 swiper
			renderSwiper(index) {

				this.pageIndex = index + 1
				console.log(index, this.dataList, '渲染 swiper------');
				// if (index + 1 >= this.count) {
				// 	uni.showToast({
				// 		title: '已是最后一题' + this.countResult.success,
				// 		icon: 'none'
				// 	})
				// 	return
				// }
				// console.log(index+1,this.dataList.length,'this.dataList.length');

				let list = [];
				let current = 1;
				if (this.dataList[index - 1]) {
					list.push(this.dataList[index - 1]);
				} else {
					current = 0;
				}

				list.push(this.dataList[index])

				if (this.dataList[index + 1]) {
					list.push(this.dataList[index + 1]);
				}

				this.duration = 0;

				setTimeout(() => {
					this.swiperList = list;
					this.swiperIndex = current;

					setTimeout(() => {
						this.duration = 200;
					}, 100)
				}, 50)
				// if (!this.examType) {
				// 	if (index + 1 == this.dataList.length) {
				// 		this.swiperTypeGet = false
				// 		this.page++
				// 		this.getList()
				// 	}
				// }

			},
			// 轮播图切换
			onChange(e) {
				console.log(e.detail.current, '轮播当前索引');
				// innerAudioContext.stop()
				// this.backMusicType = false

				// 非触摸事件不做轮播图状态更新
				if (e.detail.source != "touch") return;

				// 标识已切换
				this.isChange = true;
				// 轮播图当前位置大于原来时则表示为下一题
				if (e.detail.current > this.swiperIndex) {
					this.topicIndex++;
				} else {
					// 轮播图当前位置小于原来时则表示为上一题
					this.topicIndex--;
				}

				// 更新轮播图位置数值，为更新时让 Vue 能监听到数据有改变
				this.swiperIndex = e.detail.current;
				console.log(this.topicIndex, 'this.topicIndex');
				innerAudioContext.stop()
				innerAudioContext.src = 'http://wfqn.xllcedupro.top/' + encodeURIComponent(this.dataList[this.topicIndex]
					.audio);
				innerAudioContext.autoplay = true;
				innerAudioContext.loop = true
				innerAudioContext.obeyMuteSwitch = false;
				innerAudioContext.play();
				this.backMusicType = true
			},
			// 轮播图动画结束
			onAnimationfinish(e) {
				if (!this.isChange) return;

				this.isChange = false;
				setTimeout(() => {
					this.renderSwiper(this.topicIndex);
				}, 10);

			},
			// 选择题目
			pickerTopic(index) {
				this.topicIndex = index;
				innerAudioContext.stop()
				innerAudioContext.src = 'http://wfqn.xllcedupro.top/' + encodeURIComponent(this.dataList[this.topicIndex]
					.audio);
				innerAudioContext.autoplay = true;
				innerAudioContext.loop = true
				innerAudioContext.obeyMuteSwitch = false;
				innerAudioContext.play();
				this.backMusicType = true
				this.renderSwiper(index);
				this.showPopup = false;
			},
			// 监听答题
			onAnswerSheet(index) {
				if (this.swiperList[this.swiperIndex].answerResult) {
					// 不能重复答题
					return;
				}
				// console.log(index,this.topicIndex,this.swiperIndex)

				// 当前已选答案
				let seelctedValue = this.swiperList[this.swiperIndex].answerSheet[index].value;

				//考试模式用来判断答案是否相同计算分数  错题则进入到错题库
				// console.log(seelctedValue, this.swiperList[this.swiperIndex].answer, '当前已选答案' , '正确答案');
				this.$set(this.swiperList[this.swiperIndex], "answerResult", seelctedValue);
				if (seelctedValue.toString() != this.swiperList[this.swiperIndex].answer.toString()) {
					console.log(this.swiperList[this.swiperIndex].errAudio, '答题错误播放地址');
					innerAudioContext.stop()
					innerAudioContext.src = 'http://wfqn.xllcedupro.top/' + encodeURIComponent(this.swiperList[this
						.swiperIndex].errAudio);
					innerAudioContext.autoplay = true;
					innerAudioContext.loop = true
					innerAudioContext.obeyMuteSwitch = false;
					innerAudioContext.play();
					this.backMusicType = false
					// 记录错题接口------------------------
					// addTopicError({
					// 	topicId: this.swiperList[this.swiperIndex].topicId,
					// 	type: 1
					// }).then(res => {})
				}
				// console.log(this.swiperList[this.swiperIndex])
				// console.log(this.dataList[this.topicIndex])

				// 答题完成后自动进入下一题
				if (seelctedValue.toString() == this.swiperList[this.swiperIndex].answer.toString()) {
					innerAudioContext.stop()
					this.autoNextTopicTime = setTimeout(() => {
						let pickerIndex = this.topicIndex + 1;
						if (pickerIndex >= this.count) {
							uni.showToast({
								title: '已是最后一题' + this.countResult.success,
								icon: 'none'
							})
							return
						}
						// if (pickerIndex == this.dataList.length) {
						// 	this.page++
						// 	this.getList()
						// }
						this.pickerTopic(pickerIndex);
					}, 1200);
				}

			},


			onAnswerSheets(index) {
				if (this.swiperList[this.swiperIndex].isShow) {
					// 不能重复答题
					return;
				}


				// 当前已选答案
				let seelctedValue = this.swiperList[this.swiperIndex].answerSheet[index].value;
				if (this.answerResultString.indexOf(seelctedValue) == -1) {
					this.answerResultString.push(seelctedValue)
				} else {
					this.answerResultString = this.answerResultString.filter(item => item != seelctedValue)
				}
				//考试模式用来判断答案是否相同计算分数  错题则进入到错题库
				// console.log(seelctedValue, this.swiperList[this.swiperIndex].answer, '当前已选答案' , '正确答案');
				this.$set(this.swiperList[this.swiperIndex], "answerResult", this.answerResultString.join());
				console.log(this.swiperList[this.swiperIndex], '已选答案返回数据');


				// console.log(this.swiperList[this.swiperIndex])
				// console.log(this.dataList[this.topicIndex])



			},
			//
			btnCheckbox(index) {
				if (this.swiperList[this.swiperIndex].isShow) {
					// 不能重复答题
					return;
				}
				if (this.swiperList[this.swiperIndex].answerResult.length < 2) {
					uni.showToast({
						title: '多选题最少选择两项',
						icon: 'none'
					})
					return
				}
				this.$set(this.swiperList[this.swiperIndex], "isShow", true);
				if (this.swiperList[this.swiperIndex].answerResult.split(',').sort().toString() != this.swiperList[this
						.swiperIndex].answer.split(',').sort().toString()) {
					console.log('打错了');
					innerAudioContext.stop()
					innerAudioContext.src = 'http://wfqn.xllcedupro.top/' + encodeURIComponent(this
						.swiperList[this.swiperIndex].errAudio);
					innerAudioContext.autoplay = true;
					innerAudioContext.loop = true
					innerAudioContext.obeyMuteSwitch = false;
					innerAudioContext.play();
					this.backMusicType = false
					// addTopicError({
					// 	topicId: this.swiperList[this.swiperIndex].topicId,
					// 	type: 1
					// }).then(res => {})
				}
				// 答题完成后自动进入下一题
				if (this.swiperList[this.swiperIndex].answerResult.split(',').sort().toString() == this.swiperList[this
						.swiperIndex].answer.split(',').sort().toString()) {
					innerAudioContext.stop()
					this.autoNextTopicTime = setTimeout(() => {
						let pickerIndex = this.topicIndex + 1;
						if (pickerIndex >= this.count) {
							uni.showToast({
								title: '已是最后一题' + this.countResult.success,
								icon: 'none'
							})
							return
						}
						this.pickerTopic(pickerIndex);
					}, 1200);
				}
				// console.log(this.swiperList[this.swiperIndex], '已选答案返回数据');

			},
			// 清除自动跳到下一页延时器
			clearAutoNextTopicTime() {
				clearTimeout(this.autoNextTopicTime);
			},
			//交卷
			btnExam() {
				this.modelShow = true
				this.$refs.countDown.pause();
				console.log(this.countResult.success + this.countResult.error);
				this.percentage = ((this.countResult.success + this.countResult.error) / this.count) * 100
				this.topicPage = this.count - (this.countResult.success + this.countResult.error)

			},
			confirmExam() {
				this.modelShow = false
				this.$refs.countDown.start();
			},
			// 确认交卷
			cancelExam() {
				let result
				if (!this.countResult.success && !this.countResult.error) {
					uni.showToast({
						title: "您暂未答题,请先答题!",
						icon: "none"
					})
					return
				}
				if (this.countResult.success > 0) {
					if (this.count == 100) {
						result = this.countResult.success
					} else {
						result = this.countResult.success * 2
					}
				} else {
					result = 0
				}
				let obj = {
					success: this.countResult.success,
					error: this.countResult.error,
					results: result,
				}
				// examineStop({
				// 	examId: this.examId,
				// 	score: result,
				// }).then(res => {

				// })
				uni.redirectTo({
					url: '/pages/index/result?item=' + JSON.stringify(obj)
				});

			},
			// 倒计时结束
			changeFinish() {
				this.modelShow = true
				console.log('倒计时结束');
			},
			// 添加收藏
			addFavorite() {
				let favorite = this.dataList[this.topicIndex].isCollect;
				console.log(favorite);
				this.$set(this.dataList[this.topicIndex], "isCollect", !favorite);
				topicCollect({
					topicId: this.dataList[this.topicIndex].topicId,
					type: favorite ? 0 : 1
				}).then(res => {
					uni.showToast({
						title: res.msg,
						icon: 'none'
					})
				})
			},
			btnRemove() {
				topicEmpty().then(res => {
					uni.showToast({
						title: res.msg,
						icon: 'none'
					})
					setTimeout(() => {
						uni.reLaunch({
							url: '/pages/home/home'
						})
					})
				})
			}
		}
	}
</script>

<style lang="scss">
	@import "@/static/helang-flex.scss";

	page {
		height: 100%;
	}

	// 容器
	.container {
		height: 100%;
		background-color: #fff;
		position: relative;
		color: #333;
		font-size: 28rpx;

	}

	// 轮播盒子
	.swiper-box {
		position: absolute;
		top: 0;
		left: 0;
		right: 0;
		bottom: 100rpx;
		height: auto;
		width: auto;
		z-index: 1;
	}

	.swiper {
		height: 100%;

		.swiper-scroll {
			height: 100%;
		}

		.swiper-item {
			background-color: #fff;
			box-sizing: border-box;
			height: 100%;



			.topic-content {
				padding: 30rpx;
				font-size: 32rpx;
				color: #333;
				line-height: 1.6;

				// 题目类型标签
				.type-label {
					background-color: #0089ff;
					color: #fff;
					border-radius: 6rpx;
					padding: 4rpx 10rpx;
					font-size: 24rpx;
				}

				// 答题卡
				.answer-sheet {
					margin: 40rpx -30rpx 20rpx -30rpx;
					// border: solid 1px red;

					.item {
						padding: 20rpx 30rpx;

						&:active {
							background-color: #f3f3f3;
						}

						+.item {
							margin-top: 0;
						}

						.icon {
							margin-right: 20rpx;
							box-shadow: 0 0 5px #999;
							width: 40rpx;
							height: 40rpx;
							border-radius: 50%;
							font-size: 26rpx;

							&.success {
								background-color: #0089ff;
							}

							&.successs {
								background-color: #0089ff;
							}

							&.error {
								background-color: #f84d27;
							}
						}


					}

				}

				// 答题结果
				.answer-result {
					background-color: #f3f3f3;
					border-radius: 8rpx;
					padding: 20rpx 30rpx;
					font-size: 28rpx;
				}

				// 题目解析
				.answer-doubt {
					font-size: 28rpx;
					color: #8a6d3b;
					background-color: #fcf8e3;
					border: #faebcc solid 1px;
					margin-top: 40rpx;
					padding: 20rpx 30rpx;
					border-radius: 8rpx;
					word-break: break-all
				}
			}
		}
	}

	/deep/.u-count-down__text {
		font-size: 38rpx !important;
	}

	// 底部面板
	.panel-bottom {
		position: absolute;
		left: 0;
		bottom: 0;
		height: auto;
		width: 100%;
		z-index: 2;

		// 遮罩
		.shade {
			position: fixed;
			z-index: 9;
			background-color: rgba(33, 33, 33, 0.5);
			top: 0;
			left: 0;
			width: 100%;
			height: 100%;
		}

		// 内容
		.content {
			position: absolute;
			z-index: 10;
			bottom: 0;
			left: 0;
			width: 100%;
			height: auto;
			background-color: #fff;
		}

		// 操作栏
		.action-bar {
			height: 100rpx;
			width: 100%;
			box-sizing: border-box;
			border-top: #ddd solid 1px;
			font-size: 28rpx;
			padding: 0 20rpx;

			.success-num {
				color: #0089ff;
			}

			.error-num {
				color: #f84d27;
			}
		}

		// 题目列表
		.topic-list {
			height: 600rpx;

			.list-box {
				padding: 20rpx 20rpx 0 20rpx;
				font-size: 28rpx;
				color: #666;

				.list-item {
					padding-bottom: 20rpx;

					>view {
						width: 80rpx;
						height: 80rpx;
						background-color: #fff;
						border: #ddd solid 1px;
						border-radius: 50%;
						margin: 0 auto;

						&.active {
							border: #0089ff solid 1px;
						}

						&.success {
							background-color: #dbf2ff;
							color: #0089ff;
							border: none;

							&.active {
								border: #0089ff solid 1px;
							}
						}

						&.error {
							background-color: #ffeceb;
							color: #f84d27;
							border: none;

							&.active {
								border: #f84d27 solid 1px;
							}
						}
					}
				}
			}
		}
	}
</style>
