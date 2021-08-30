<template>
	<view v-show="value" class="calendar-body">
		<view class="title-name">
			{{title}} <text class="iconfont icon-close-bold"  @click="close()"></text>
		</view>
		<view v-if="mode == 'range'" class="range-text">
			<view class="left">
				<view class="alt">入住日期</view>
				<view class="time">
					<text>{{startMonth}}月{{startDay}}日</text> {{startWeek}}
				</view>
			</view>
			<view class="right">
				<view class="alt">离店日期</view>
				<view class="time">
					<text>{{endMonth}}月{{endDay}}日</text> {{endWeek}}
				</view>
			</view>
		</view>
		<scroll-view  class="day-list" :class="{'day-range': mode == 'range'}" scroll-y="true" :scroll-top="scrollTop" @scrolltolower="changeMonthHandler(1)" style="width: 100%;">
			<view class="day-item" v-for="item in listData" :key="item.showTitle">
				<view class="day-title">{{item.showTitle}}</view>
				<view class="week-list">
					<view class="week-day__text" v-for="(itemZh, indexZh) in weekDayZh" :key="indexZh">{{itemZh}}</view>
				</view>
				
				<view class="content">
					<!-- 前置空白部分 -->
					<block v-for="(itemArr, indexArr) in item.weekdayArr" :key="indexArr">
						<view  class="item" ></view>
					</block>
					<view  class="item" v-for="(dayItem, dayIndex) in item.daysArr" :key="dayIndex"
					:class="{ 
						hoverData: openDisAbled(item.year, item.month, dayIndex+1),
						activeData: mode== 'date' && activeDate ==`${item.year}-${item.month}-${dayIndex+1}`,
						startDate: mode == 'range' && startDate==`${item.year}-${item.month}-${dayIndex+1}`,
						startOrend: mode == 'range' && openStartOrend(item.year, item.month, dayIndex+1),
						endDate: mode== 'range' && endDate==`${item.year}-${item.month}-${dayIndex+1}`
					}"
					@tap="dateClick(item, dayIndex)">
						<view class="number">{{ dayIndex + 1 }}</view>
						<view class="dec">{{ viewDayAlt(item.year, item.month, dayIndex+1) }}</view>
					</view>
				</view>
			</view>
		</scroll-view>
	</view>
</template>
<script>
	/**
	 * calendar 日历
	 */
	
	export default {
		name: 'calendar',
		props: {
			// 通过双向绑定控制组件的弹出与收起
			value: {
				type: Boolean,
				default: false
			},
			// date-单个日期选择，range-开始日期+结束日期选择
			mode: {
				type: String,
				default: 'date'
			},
			// 日期选择标题
			title: {
				type: String,
				default: '可预约日期查询'
			},
			// 最小下单日期(不在范围内日期禁用不可选，包含当天)
			minDate: {
				type: [Number, String],
				default: '1950-01-01'
			},
			// 最大下单日期(不在范围内日期禁用不可选，包含当天)
			maxDate: {
				type: [Number, String],
				default: '2200-12-31'
			},
			// 是否禁止周六周日 不可下单 默认不禁止
			isWeekend: {
				type: Boolean,
				default: false
			},
			// 每天显示的提示
			altPrice: {
				type: Array,
				default: [{
					date: '1950-01-01',
					price: ''
				}]
			},
			// 入住离开日期
			addOrRemoveData: {
				type: Array,
				default: () => { return [] }
			}
		},
		data() {
			return {
				scrollTop: 0,
				datePrice: [],
				// 星期几,值为1-7
				weekday: 1, 
				weekdayArr:[],
				// 当前月有多少天
				days: 0, 
				daysArr:[],
				showTitle: '',
				year: 2020,
				month: 0,
				day: 0,
				startYear: 0,
				startMonth: 0,
				startDay: 0,
				startWeek: '',
				endYear: 0,
				endMonth: 0,
				endDay: 0,
				endWeek: '',
				today: '',
				activeDate: '',
				startDate: '',
				endDate: '',
				isStart: true,
				min: null,
				max: null,
				weekDayZh: ['日', '一', '二', '三', '四', '五', '六']
			};
		},
		computed: {
			dataChange() {
				return `${this.mode}-${this.minDate}-${this.maxDate}`;
			}
		},
		watch: {
			dataChange(val) {
				this.init()
			}
		},
		created() {
			this.init()
		},
		methods: {
			init() {
				// 初始化日期下的 文字
				this.datePrice = this.altPrice
				/// 初始化日期
				let now = new Date();
				this.year = now.getFullYear();
				this.month = now.getMonth() + 1;
				this.day = now.getDate();
				this.today = `${now.getFullYear()}-${now.getMonth() + 1}-${now.getDate()}`;
				this.activeDate = this.today;
				this.min = this.initDate(this.minDate);
				this.max = this.initDate(this.maxDate || this.today);
				this.startDate = "";
				this.startWeek = "";
				this.startYear = 0;
				this.startMonth = 0;
				this.startDay = 0;
				this.endYear = 0;
				this.endMonth = 0;
				this.endDay = 0;
				this.endDate = "";
				this.endWeek = "";
				this.isStart = true;
				this.listData = []
				this.changeData();
				// 默认显示两个月
				let _this = this
				setTimeout(() => {
					_this.changeMonthHandler(1)
				}, 100)
				
				// 初始化入住离店时间
				if (this.addOrRemoveData.length > 0) {
					setTimeout(() => {
						_this.addOrRemoveChange()
					}, 200)
				}
			},
			//日期处理
			initDate(date) {
				let fdate = date.split('-');
				return {
					year: Number(fdate[0] || 1920),
					month: Number(fdate[1] || 1),
					day: Number(fdate[2] || 1)
				}
			},
			// 判断可使用日期范围
			openDisAbled(year, month, day) {
				let bool = true;
				let date = `${year}/${month}/${day}`;
				let today = this.today.replace(/\-/g, '/');
				let min = `${this.min.year}/${this.min.month}/${this.min.day}`;
				let max = `${this.max.year}/${this.max.month}/${this.max.day}`;
				let timestamp = new Date(date).getTime();
				if (timestamp < new Date(min).getTime() || timestamp > new Date(max).getTime()) {
					bool = false;
				}
				// 判断是否大于今日
				if (timestamp < new Date(today).getTime()) {
					bool = false;
				}
				
				// 禁止周六周日下单
				if (this.isWeekend && (this.getWeekText(date) === '星期六' || this.getWeekText(date) === '星期日')) {
					bool = false;
				}
				
				
				
				return bool;
			},
			// 判断是否再选中日期范围内
			openStartOrend(year, month, day) {
				let bool = false;
				let date = `${year}/${month}/${day}`;
				let startDate = this.startDate.replace(/\-/g, '/');
				let endDate = this.endDate.replace(/\-/g, '/');
				let timestamp = new Date(date).getTime();
				
				if (timestamp > new Date(startDate).getTime() && timestamp < new Date(endDate).getTime()) {
					bool = true
				}
				
				return bool
			},
			// 显示日期下的文字提示
			viewDayAlt(year, month, day) {
				let txt = ''
				let item = this.datePrice.find(item => {
					let today = item.date.replace(/\-/g, '/');
					let date = `${year}/${month}/${day}`;
					return new Date(date).getTime() === new Date(today).getTime()
				})
				if (item) {
					return Number.isFinite(item.price) ? '￥' + item.price : item.price
				} 
			},
			generateArray(start, end) {
				return Array.from(new Array(end + 1).keys()).slice(start);
			},
			formatNum(num) {
				return num < 10 ? '0' + num : num + '';
			},
			//一个月有多少天
			getMonthDay(year, month) {
				let days = new Date(year, month, 0).getDate();
				return days;
			},
			getWeekday(year, month) {
				let date = new Date(`${year}/${month}/01 00:00:00`);
				return date.getDay();
			},
			checkRange(year) {
				let overstep = false;
				if (year < this.minYear || year > this.maxYear) {
					uni.showToast({
						title: "日期超出范围啦~",
						icon: 'none'
					})
					overstep = true;
				}
				return overstep;
			},
			changeMonthHandler(isAdd) {
				if (isAdd) {
					// 根据日期限制显示
					if (this.year >= this.max.year && this.month >= this.max.month) {
						return false
					}
					
					let month = this.month + 1;
					let year = month > 12 ? this.year + 1 : this.year;
					if (!this.checkRange(year)) {
						this.month = month > 12 ? 1 : month;
						this.year = year;
						this.changeData();
					}

				} else {
					let month = this.month - 1;
					let year = month < 1 ? this.year - 1 : this.year;
					if (!this.checkRange(year)) {
						this.month = month < 1 ? 12 : month;
						this.year = year;
						this.changeData();
					}
				}
			},
			changeYearHandler(isAdd) {
				let year = isAdd ? this.year + 1 : this.year - 1;
				if (!this.checkRange(year)) {
					this.year = year;
					this.changeData();
				}
			},
			changeData() {
				this.days = this.getMonthDay(this.year, this.month);
				this.daysArr=this.generateArray(1,this.days)
				this.weekday = this.getWeekday(this.year, this.month);
				this.weekdayArr=this.generateArray(1,this.weekday)
				this.showTitle = `${this.year}年${this.month}月`;
				if (this.isChange && this.mode == 'date') {
					this.btnFix(true);
				}
				this.setDayText()
			},
			// 处理插入数据时间列表
			setDayText() {
				this.listData.push({
					year: this.year,
					month: this.month,
					weekdayArr: this.weekdayArr,
					daysArr: this.daysArr,
					showTitle: this.showTitle
				})
			},
			// 处理初始化入住离店时间
			addOrRemoveChange() {
				let _this = this
				this.addOrRemoveData.forEach((item, index) => {
					let data = item.replace(/\-/g, '/')
					let now = new Date(data);
					let year = now.getFullYear();
					let month = now.getMonth() + 1;
					let day = now.getDate() - 1;
					
					if (index === 0) {
						this.dateClick({year: year, month: month }, day)
					} else {
						setTimeout(() => {
							_this.dateClick({year: year, month: month }, day)
						}, 300)
					}
				})
			},
			// 选择日期操作
			dateClick(item, day) {
				day += 1;
				if (this.openDisAbled(item.year, item.month, day)) {
					this.day = day;
					let date = `${item.year}-${item.month}-${day}`;
					if (this.mode == 'date') {
						this.activeDate = date;
						this.btnFix(true)
					} else {
						let compare = new Date(date.replace(/\-/g, '/')).getTime() < new Date(this.startDate.replace(/\-/g, '/')).getTime()
						if (this.isStart || compare) {
							this.startDate = date;
							this.startYear = item.year;
							this.startMonth = item.month;
							this.startDay = this.day;
							this.endYear = 0;
							this.endMonth = 0;
							this.endDay = 0;
							this.endDate = "";
							this.activeDate = "";
							this.startWeek = this.getWeekText(this.startDate);
							this.isStart = false;
							// 文字提示
							this.datePrice = [{
								date: date,
								price: '入住'
							}]
						} else {
							this.endDate = date;
							this.endYear = item.year;
							this.endMonth = item.month;
							this.endDay = this.day;
							this.endWeek = this.getWeekText(this.endDate);
							this.isStart = true;
							// 文字提示
							this.datePrice.push({
								date: date,
								price: '离开'
							})
							this.btnFix(true)
						}
					}
				}
			},
			close() {
				this.scrollTop = 0
				// 修改通过v-model绑定的父组件变量的值为false，从而隐藏日历弹窗
				this.$emit('input', false);
			},
			getWeekText(date) {
				date = new Date(`${date.replace(/\-/g, '/')} 00:00:00`);
				let week = date.getDay();
				return '星期' + ['日', '一', '二', '三', '四', '五', '六'][week];
			},
			btnFix(show) {
				if (!show) {
					this.close();
				}
				if (this.mode == 'date') {
					let arr = this.activeDate.split('-')
					let year = this.isChange ? this.year : Number(arr[0]);
					let month = this.isChange ? this.month : Number(arr[1]);
					let day = this.isChange ? this.day : Number(arr[2]);
					//当前月有多少天
					let days = this.getMonthDay(year, month);
					let result = `${year}-${this.formatNum(month)}-${this.formatNum(day)}`;
					let weekText = this.getWeekText(result);
					let isToday = false;
					if (`${year}-${month}-${day}` == this.today) {
						//今天
						isToday = true;
					}
					this.$emit('change', {
						year: year,
						month: month,
						day: day,
						days: days,
						activeDate: this.activeDate,
						result: result,
						week: weekText,
						isToday: isToday
					});
				} else {
					if (!this.startDate || !this.endDate) return;
					let startMonth = this.formatNum(this.startMonth);
					let startDay = this.formatNum(this.startDay);
					let startDate = `${this.startYear}-${startMonth}-${startDay}`;
					let startWeek = this.getWeekText(startDate)

					let endMonth = this.formatNum(this.endMonth);
					let endDay = this.formatNum(this.endDay);
					let endDate = `${this.endYear}-${endMonth}-${endDay}`;
					let endWeek = this.getWeekText(endDate);
					this.$emit('change', {
						startYear: this.startYear,
						startMonth: this.startMonth,
						startDay: this.startDay,
						startDate: startDate,
						startWeek: startWeek,
						endYear: this.endYear,
						endMonth: this.endMonth,
						endDay: this.endDay,
						endDate: endDate,
						endWeek: endWeek
					});
				}
			}
		}
	};
</script>

<style scoped lang="scss">
$c_01: #008CCE;
$c_02: #DADADA;
$c_03: #96292F;
$c_04: #FAFAFA;
$c_05: #5DC0EF;
$c_06: #ED7444;
.calendar-body{
	position: fixed;
	top: 0;
	left: 0;
	bottom: 0;
	right: 0;
	overflow: hidden;
	background-color: $c_04;
	animation: fadelogIn .4s;
	z-index: 9;
	.title-name{
		font-size: 36upx;
		font-weight: 500;
		line-height: 48upx;
		display: flex;
		align-items: center;
		justify-content: center;
		position: relative;
		padding: 24upx;
		background-color: $c_04;
		.icon-close-bold{
			position: absolute;
			top: 24upx;
			right: 24upx;
			width: 40upx;
			height: 40upx;
			font-size: 40upx;
			color: #999999;
		}
	}
	.range-text{
		display: flex;
		align-items: center;
		justify-content: space-between;
		padding: 0 24upx;
		.alt{
			font-size: 24upx;
			color: #303030;
		}
		.time{
			font-size: 24upx;
			color: #666666;
			text{
				font-size: 36upx;
				color: #000000;
			}
		}
		.right{
			text-align: right;
		}
	}
	.day-list{
		padding: 24upx;
		height: calc(100vh - 100upx);
		box-sizing: border-box;
		.day-item{
			background: #FFFFFF;
			border-radius: 16upx;
			margin-bottom: 24upx;
		}
		.day-title{
			font-size: 30upx;
			font-weight: 500;
			color: #000000;
			text-align: center;
			padding: 30upx 0;
		}
		.week-list{
			display: flex;
			border-bottom: 1upx solid #F0F0F0;
			padding: 24upx;
			.week-day__text{
				flex: 1;
				text-align: center;
				font-size: 24upx;
				color: #000000;
				width: 14.28%;
				&:last-child{
					color: $c_06;
				}
				&:first-child{
					color: $c_06;
				}
			}
		}
		.content{
			display: flex;
			flex-wrap: wrap;
			padding: 24upx;
			.item{
				width: 14.28%;
				height: 90upx;
				border-radius: 8upx;
				display: flex;
				align-items: center;
				justify-content: center;
				flex-direction: column;
				margin-bottom: 24upx;
				.number{
					font-size: 24upx;
					color: #999999;
				}
				.dec{
					font-size: 24upx;
					font-weight: bold;
					color: $c_06;
					min-height: 30upx;
					opacity: 0;
				}
			}
			.hoverData{
				.number{
					color: #333333;
				}
				.dec{
					opacity: 1;
				}
			}
			.activeData{
				background: $c_03;
				border-radius: 8upx;
				.number{
					color: #FFFFFF;
				}
				.dec{
					color: #FFFFFF;
				}
			}
			.startDate{
				background: $c_03;
				border-radius: 8upx;
				border-top-right-radius: 0;
				border-bottom-right-radius: 0;
				.number{
					color: #FFFFFF;
				}
				.dec{
					color: #FFFFFF;
				}
			}
			.endDate{
				background: $c_03;
				border-radius: 8upx;
				border-top-left-radius: 0;
				border-bottom-left-radius: 0;
				.number{
					color: #FFFFFF;
				}
				.dec{
					color: #FFFFFF;
				}
			}
			.startOrend{
				background-color: #F4EAEB;
				border-radius: 0
			}
		}
	}
	.day-range{
		height: calc(100vh - 180upx);
	}
}

@keyframes fadelogIn {
    0% {
        transform: translate3d(0, 100%, 0);
    }
    100% {
        transform: none;
    }
}
</style>