<template>
	<view class="ruler-wrap">
		<!-- 日期选择 -->
		<DateOfMenstrualPeriod 
		mode="range"
		:value="ovulatoryPeriod"
		@change="changeDate"></DateOfMenstrualPeriod>
		
		
		
		<view class="cur-value">肉类卡路里：{{foodkg}} -- {{kcal}}</view>
		<ZxRuler
		 CTRLID="heightRuler"
		 :min="20"
		 :max="230"
		 :rulerWidth="600"
		 :rulerHeight="120"
		 orientation="horizontal"
		 v-model="mainValue"
		 @onScrollChange="getRulerValue"></ZxRuler>
		 
		 <ZxRuler
		  CTRLID="weightRuler"
		  :min="20"
		  :max="230"
		  :rulerWidth="120"
		  :rulerHeight="480"
		  orientation="vertical"
		  v-model="mainValue"
		  @onScrollChange="getRulerValue"></ZxRuler>
	</view>
</template>

<script>
	import DateOfMenstrualPeriod from '../../components/DateOfMenstrualPeriod.vue'
	import ZxRuler from '../../components/ZxRuler.vue'
	export default {
		components:{
			ZxRuler,
			DateOfMenstrualPeriod
		},
		data() {
			return {
				showCalendar: true,
				dataPrice: [ // 日历上的价格
					{
						date: '2021-9-23',
						price: 98
					},
					{
						date: '2021-9-28',
						price: 108
					},
					{
						date: '2021-9-29',
						price: 98
					}
				],
				ovulatoryPeriod: {},
				mainValue: 100,
				kcal: 0,
				foodkg: 0
			}
		},
		methods: {
			getRulerValue(result){
				this.foodkg = result;
				this.kcal = (result*8.2).toFixed(1);
			},
			// 获取选中的日期
			changeDate(data) {
				console.log(data)
			}
		},
	}
</script>

<style>
	.ruler-wrap{
		display: inline-block;
		margin: 30rpx;
		text-align: center;
	}
	.cur-value{
		text-align: center;
		font-size: 16px;
	}
</style>
