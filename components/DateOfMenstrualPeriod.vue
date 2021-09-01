<template>
	<view class="calendar-body">
		<view  class="day-list">
			<view class="day-title"> 
				<button type="default" size="mini" @click="changeMonthHandler(false)">上月</button>
				{{curDateRange.showTitle}} 
				<button type="default" size="mini" @click="changeMonthHandler(true)">下月</button>
			</view>
			<view class="week-list">
				<view class="week-day__text" v-for="(itemZh, indexZh) in weekDayZh" :key="indexZh">{{itemZh}}</view>
			</view>
			
			<view class="content">
				<!-- 前置空白部分 -->
				<block v-for="(itemArr, indexArr) in curDateRange.weekdayArr" :key="indexArr">
					<view class="item"></view>
				</block>
				<view :class="['item', { 
				hoverData: openDisabled(curDateRange.year, curDateRange.month, dayIndex+1),
				activeData: mode == 'date' && activeDate ==`${curDateRange.year}-${curDateRange.month}-${dayIndex+1}`,
				startDate: mode == 'range' && startDate == `${curDateRange.year}-${curDateRange.month}-${dayIndex+1}`,
				startOrend: mode == 'range' && isMenstrualPeriod(curDateRange.year, curDateRange.month, dayIndex+1),
				endDate: mode == 'range' && endDate == `${curDateRange.year}-${curDateRange.month}-${dayIndex+1}`,
				ovulatoryDate: mode == 'range' && isOvulation(curDateRange.year, curDateRange.month, dayIndex+1)
				}]" v-for="(dayItem, dayIndex) in curDateRange.daysArr" :key="dayIndex"
				@tap="dateClick(curDateRange, dayIndex)">
					<view class="number">{{ dayIndex + 1 }}</view>
				</view>
			</view>
		</view>
	</view>
</template>
<script>
	export default {
		name: 'DateOfMenstrualPeriod',			//月经周期日期组件
		props: {
			// 通过双向绑定控制组件的弹出与收起
			value: {
				type: Object,
				default: () => {
					return {}
				}
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
			// 入住离开日期
			addOrRemoveData: {
				type: Array,
				default: () => { 
					return [] 
				}
			}
		},
		data() {
			return {
				curDateRange: {},
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
				weekDayZh: ['日', '一', '二', '三', '四', '五', '六'],
				ovulatoryPeriod: []
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
				/// 初始化日期
				let now = new Date();
				this.year = now.getFullYear();
				this.month = now.getMonth() + 1;
				this.day = now.getDate();
				this.today = `${now.getFullYear()}-${now.getMonth() + 1}-${now.getDate()}`;
				this.activeDate = this.today;
				this.min = this.initDate(this.minDate);
				this.max = this.initDate(this.maxDate || this.today);
				this.startDate = this.ovulatoryPeriod[0] || "";
				this.startWeek = "";
				this.startYear = 0;
				this.startMonth = 0;
				this.startDay = 0;
				this.endYear = 0;
				this.endMonth = 0;
				this.endDay = 0;
				this.endDate = this.ovulatoryPeriod[1] || "";
				this.endWeek = "";
				this.isStart = true;
				this.curDateRange = {};
				this.changeData();
				// 默认显示两个月
				let _this = this
				// setTimeout(() => {
				// 	_this.changeMonthHandler(1)
				// }, 100)
				
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
			openDisabled(year, month, day) {
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
			// 判断是否在月经期
			isMenstrualPeriod(year, month, day) {
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
			// 判断是否在排卵期
			isOvulation(year, month, day) {
				let bool = false;
				let date = `${year}/${month}/${day}`;
				let startDate = this.startDate.replace(/\-/g, '/');
				let endDate = this.endDate.replace(/\-/g, '/');
				let timestamp = new Date(date).getTime();
				if (timestamp >= new Date(startDate).getTime() && timestamp <= new Date(endDate).getTime()) {
					bool = true
				}				
				return bool
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
				this.weekdayArr=this.generateArray(1,this.weekday);
				this.showTitle = `${this.year}年${this.month}月`;
				if (this.isChange && this.mode == 'date') {
					this.sendCtrlData();
				}

				this.curDateRange = {
					year: this.year,
					month: this.month,
					weekdayArr: this.weekdayArr,
					daysArr: this.daysArr,
					showTitle: this.showTitle
				}
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
				if (this.openDisabled(item.year, item.month, day)) {
					this.day = day;
					let date = `${item.year}-${item.month}-${day}`;
					if (this.mode == 'date') {
						this.activeDate = date;
						this.sendCtrlData()
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
						} else {
							this.endDate = date;
							this.endYear = item.year;
							this.endMonth = item.month;
							this.endDay = this.day;
							this.endWeek = this.getWeekText(this.endDate);
							this.isStart = true;
							this.sendCtrlData()
						}
					}
				}
			},
			getWeekText(date) {
				date = new Date(`${date.replace(/\-/g, '/')} 00:00:00`);
				let week = date.getDay();
				return '星期' + ['日', '一', '二', '三', '四', '五', '六'][week];
			},
			sendCtrlData() {
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
$c_03: #FF9C9D;
$c_04: #FAFAFA;
$c_05: #5DC0EF;
$c_06: #ED7444;
.day-list{
	width: 100%;
	background: #FFFFFF;
	border-radius: 16rpx;
	box-sizing: border-box;
	.day-title{
		font-size: 30rpx;
		font-weight: 500;
		color: #000000;
		display: flex;
		flex-direction: row;
		justify-content: center;
		align-items: center;
		padding: 30rpx 0;
		border-bottom: 1rpx solid #F0F0F0;
	}
	.week-list{
		display: flex;
		padding: 24rpx;
		.week-day__text{
			flex: 1;
			text-align: center;
			font-size: 24rpx;
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
		padding: 0 24rpx 24rpx;
		.item{
			width: 14.28%;
			height: 80rpx;
			border-radius: 8rpx;
			display: flex;
			align-items: center;
			justify-content: center;
			flex-direction: column;
			margin-bottom: 20rpx;
			.number{
				width: 80rpx;
				height: 80rpx;
				display: flex;
				justify-content: center;
				align-items: center;
				font-size: 24rpx;
				color: #999999;
			}
		}
		.hoverData{
			.number{
				color: #333333;
			}
		}
		.activeData{
			background: $c_02;
			border-radius: 90rpx;
			.number{
				color: #FFFFFF;
			}
		}
		.startDate{
			background: $c_03;
			border-radius: 90rpx;
			border-top-right-radius: 0;
			border-bottom-right-radius: 0;
			.number{
				color: #FFFFFF;
			}
		}
		.endDate{
			background: $c_03;
			border-radius: 90rpx;
			border-top-left-radius: 0;
			border-bottom-left-radius: 0;
			.number{
				color: #FFFFFF;
			}
		}
		.startOrend{
			background-color: $c_03;
			border-radius: 0;
			.number{
				color: #FFFFFF;
			}
		}
		.ovulatoryDate{
			position: relative;
			&:after{
				width: 10rpx;
				height: 10rpx;
				content: '';
				background-color: $c_05;
				border-radius: 50%;
				position: absolute;
				left: 50%;
				bottom: 10rpx;
				transform: translateX(-50%);
			}
		}
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