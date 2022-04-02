<template>
	<view class="bg-white">
		<view class="flex justify-center padding">
			<text class="text-df ">{{curDayStr}}</text>
		</view>
		<view class="flex justify-center align-center text-xxl text-bold padding">
			<text class="cuIcon-back padding-right" @tap="subDay"></text>
			<picker mode="date" @change="dateChange">
				<text class="text-black text-shadow">{{dayStr}}</text>
			</picker>
			<text class="cuIcon-right padding-left" @tap="addDay"></text>
		</view>
		<view class="flex justify-around">
			<text>{{yearStr}}</text>
			<text class="text-bold">{{weekStr}}</text>
		</view>
		<view class="padding-xl">
			<view class="flex align-center padding-bottom-xs">
				<text class="cu-tag round bg-green ">宜</text>
				<text class="text-df padding-left-xs">{{yi}}</text>
			</view>
			<view class="flex align-center ">
				<text class="cu-tag round bg-red ">忌</text>
				<text class="text-df padding-left-xs">{{ji}}</text>
			</view>
		</view>
		<view class="flex justify-around padding-top-xl padding-left-xl padding-right-xl">
			<view class="padding flex flex-direction align-center ">
				<view class="text-xl text-red text-shadow padding-bottom-sm">喜神</view>
				<view class="text-lg">{{xi}}</view>
			</view>
			<view class="padding flex flex-direction align-center ">
				<view class="text-xl text-red text-shadow padding-bottom-sm">福神</view>
				<view class="text-lg">{{fu}}</view>
			</view>
			<view class="padding flex flex-direction align-center ">
				<view class="text-xl text-red text-shadow padding-bottom-sm">财神</view>
				<view class="text-lg">{{cai}}</view>
			</view>
		</view>
		<view class="flex justify-center padding-bottom">
			<view class="padding flex flex-direction align-center ">
				<view class="text-xl text-red text-shadow padding-bottom-sm">阳贵神</view>
				<view class="text-lg">{{yanggui}}</view>
			</view>
			<view class="padding flex flex-direction align-center ">
				<view class="text-xl text-red text-shadow padding-bottom-sm">阴贵神</view>
				<view class="text-lg">{{yingui}}</view>
			</view>
		</view>
		
		<view class="padding-xl">
			<view class="flex align-center padding-bottom">
				<text class="text-lg padding-right text-bold text-shadow text-black">星宿</text>
				<text>{{xing}}</text>
			</view>
			<view class="flex align-center padding-bottom">
				<text class="text-lg padding-right text-bold text-shadow text-black">冲煞</text>
				<text>{{sha}}</text>
			</view>
			<view class="flex align-center padding-bottom">
				<text class="text-lg padding-right text-bold text-shadow text-black">彭祖</text>
				<text>{{peng}}</text>
			</view>
		</view>
	</view>
</template>

<script>
	const {
		Solar,
		SolarMonth,
		Lunar,
		HolidayUtil
	} = require('@/common/lunar.js');
	export default {
		data() {
			return {
				curDay: new Date(),
				curDayStr: '',
				dayStr: '',
				yearStr: '',
				weekStr: '',
				ji: '',
				yi: '',
				xi: '',
				fu: '',
				cai: '',
				gui: '',
				yanggui:'',
				yingui:'',
				xing: '',
				sha: '',
				peng: '',
			}
		},
		created() {
			this.render();
		},
		methods: {
			render(setDayStr) {
				this.curDay = setDayStr ? new Date(setDayStr) : new Date();
				let d = Lunar.fromDate(this.curDay);
				// console.log(d.toFullString())
				let sd = d.getSolar();
				this.curDayStr = sd.getYear() + '年' + sd.getMonth() + '月' + sd.getDay() + '日';
				this.dayStr = d.getMonthInChinese() + '月' + d.getDayInChinese();
				this.yearStr = d.getYearInGanZhi() + '(' + d.getYearShengXiao() + ')年';
				this.weekStr = '星期' + d.getWeekInChinese();
				this.ji = d.getDayJi().join(' ');
				this.yi = d.getDayYi().join(' ');
				this.xi = d.getDayPositionXiDesc();
				this.fu = d.getDayPositionFuDesc();
				this.cai = d.getDayPositionCaiDesc();
				this.gui = '阳神' + d.getPositionYangGuiDesc() + ' 阴神' + d.getPositionYinGuiDesc();
				this.yanggui = d.getPositionYangGuiDesc();
				this.yingui = d.getPositionYinGuiDesc();
				this.xing = d.getGong() + '方' + d.getXiu() + d.getZheng() + d.getAnimal() + '(' + d.getXiuLuck() + ')';
				this.sha = d.getDayShengXiao() + '日 冲' + d.getDayChongDesc() + ' ' + d.getDaySha();
				this.peng = d.getPengZuGan() + ' ' + d.getPengZuZhi();
				
				console.log('喜神：' + this.xi)
				console.log('福神：' + this.fu)
				console.log('财神：' + this.cai)
				console.log('贵神：' + this.gui)
				console.log('星宿:' + this.xing);
				console.log('冲煞：' + this.sha);
				console.log('彭祖：' + this.peng);
				console.log("节气：" + d.getJieQi())
				var p = d.getPrevJieQi(),
					n = d.getNextJieQi();
				console.log('上个节气：' + p.getName() + ' ' + p
					.getSolar().toYmdHms() + ' 星期' + p.getSolar().getWeekInChinese());
				console.log('下个节气：' + n
					.getName() + ' ' + n.getSolar().toYmdHms() + ' 星期' + n.getSolar().getWeekInChinese());
			},
			dateChange(e){
				let selectDate = undefined;
				if(e.type == 'change'){
					selectDate = new Date(e.detail.value); 
				}
				if(!selectDate){
					return;
				}
				this.render(selectDate);
			},
			subDay(){
				let d = Solar.fromDate(this.curDay);
				d = d.next(-1);
				this.render(d.toYmd());
			},
			addDay(){
				let d = Solar.fromDate(this.curDay);
				d = d.next(1);
				this.render(d.toYmd());
			},
			onShareAppMessage(res) {
			    if (res.from === 'button') {// 来自页面内分享按钮
			      console.log(res.target)
			    }
			    return {
			      title: '今日黄历',
			      path: '/pages/almanac/almanac',
				  mpId: 'wxee05508955140bfd'
			    }
			},
			onShareTimeline(res){
				  if (res.from === 'button') {// 来自页面内分享按钮
					console.log(res.target)
				  }
				  return {
					title: '今日黄历',
					path: '/pages/almanac/almanac',
					mpId: 'wxee05508955140bfd'
				  }
			},
		},
	}
</script>

<style>

</style>
