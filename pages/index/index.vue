<template>
	<view class="content">
		<Calendar  @dayChange='dayChange' @monthChange='monthChange' >
		</Calendar>
		<view class="bg-white padding " style="margin-top: 10rpx;">
			<view class="flex justify-between solid-bottom padding padding-bottom align-center">
				<view class="text-xl text-bold padding bg-blue shadow round light solid-right"><view class="text-red">农</view><view class="text-red">历</view></view>
				<view class="text-xxl text-bold padding text-shadow text-black">{{dayStr}}</view>
				<view class=" padding flex flex-direction align-center solid-left">
					<view class="text-bold text-df padding-bottom-xs">{{weekStr}}</view>
					<view class="text-yellow text-lg">{{yearStr}}</view>
				</view>
			</view>
			<view class="padding">
				<view class="flex align-center cu-card padding-bottom-xs">
					<view class="cu-tag round bg-green">宜</view>
					<view class="text-gray text-content text-sm margin-left">{{yi}}</view>
				</view>
				<view class="flex align-center cu-card ">
					<view class="cu-tag round bg-red ">忌</view>
					<view class="text-gray text-content text-sm margin-left">{{ji}}</view>
				</view>
			</view>
		</view>
	</view>
</template>

<script>
	import Calendar from '@/components/Li-Calendar/Li-Calendar.vue';
	export default {
		components: {
			Calendar
		},
		data() {
			return {
				selectDay: '',
				dayStr:'',
				weekStr:'',
				yearStr:'',
				yi:'',
				ji:'',
			}
		},
		created() {
			let curDate = new Date();
			this.selectDay =  curDate.getFullYear() + "/" + (curDate.getMonth() + 1) + "/" + curDate.getDate();
			this.render();
		},
		onLoad() {
	
		},
		methods: {
			render(setDateStr) { 
				console.info("开始进行首页下数据渲染:"+setDateStr)
				let dd = setDateStr?new Date(setDateStr):new Date(this.selectDay);
				let d = Lunar.fromDate(dd);
				this.dayStr = d.getMonthInChinese() + '月' + d.getDayInChinese();
				this.yearStr = d.getYearInGanZhi() + '(' + d.getYearShengXiao() + ')年';
				this.weekStr = '星期' + d.getWeekInChinese();
				this.ji = d.getDayJi().join(' ');
				this.yi = d.getDayYi().join(' ');
				
			},
			dayChange(data) {
				this.selectDay = data.time;
				this.render(data.time);
				console.log("日期选择变更",data)
			},
			monthChange(data) {
				console.log("月份选择变更",data)
			},
			onShareAppMessage(res) {
			    if (res.from === 'button') {// 来自页面内分享按钮
			      console.log(res.target)
			    }
			    return {
			      title: '假期日历',
			      path: '/pages/index/index',
				  mpId: 'wxee05508955140bfd'
			    }
			},
			onShareTimeline(res){
				  if (res.from === 'button') {// 来自页面内分享按钮
					console.log(res.target)
				  }
				  return {
					title: '假期日历',
					path: '/pages/index/index',
					mpId: 'wxee05508955140bfd'
				  }
			},
		}
	}
	const {Solar,SolarMonth,Lunar,HolidayUtil} = require('@/common/lunar.js');
	
	
	
</script>

<style>
	
</style>
