<template>
	<view class="">
		<view class="cur-value">当前值：{{mainValue}}</view>
		<ZxRuler
		 CTRLID="heightRuler"
		 :min="100"
		 :max="230"
		 v-model="mainValue"></ZxRuler>
	</view>
</template>

<script>
	import ZxRuler from '../../components/ZxRuler.vue'
	export default {
		components:{
			ZxRuler
		},
		data() {
			return {
				mainValue: 162
			}
		}
	}
</script>

<style>
	.ruler-wrap{
		display: inline-block;
		margin: 30rpx;
	}
	.cur-value{
		text-align: center;
		font-size: 16px;
	}
</style>
