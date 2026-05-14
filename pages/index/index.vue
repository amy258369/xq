<template>
	<view class="container">
		<view class="header">
			<view class="title-section">
				<text class="main-title">天天象棋</text>
				<text class="sub-title">经典国粹 以棋会友</text>
			</view>
			<view class="logo">
				<text class="logo-text">象</text>
			</view>
		</view>
		
		<view class="menu-section">
			<view class="menu-item" @click="startLocalBattle">
				<view class="menu-icon local-icon">
					<text class="icon-text">对</text>
				</view>
				<view class="menu-info">
					<text class="menu-title">双人对战</text>
					<text class="menu-desc">本地双人对弈</text>
				</view>
				<view class="menu-arrow">→</view>
			</view>
			
			<view class="menu-item" @click="startOnlineBattle">
				<view class="menu-icon online-icon">
					<text class="icon-text">联</text>
				</view>
				<view class="menu-info">
					<text class="menu-title">联机对战</text>
					<text class="menu-desc">好友联网对弈</text>
				</view>
				<view class="menu-arrow">→</view>
			</view>
			
			<view class="menu-item" @click="goToRanking">
				<view class="menu-icon ranking-icon">
					<text class="icon-text">榜</text>
				</view>
				<view class="menu-info">
					<text class="menu-title">积分榜</text>
					<text class="menu-desc">查看排名榜</text>
				</view>
				<view class="menu-arrow">→</view>
			</view>
			
			<view class="menu-item" @click="goToMine">
				<view class="menu-icon mine-icon">
					<text class="icon-text">我</text>
				</view>
				<view class="menu-info">
					<text class="menu-title">我的</text>
					<text class="menu-desc">个人中心</text>
				</view>
				<view class="menu-arrow">→</view>
			</view>
		</view>
		
		<view class="online-status" v-if="onlineStatus">
			<view class="status-dot" :class="{ online: isOnline }"></view>
			<text class="status-text">{{ isOnline ? '网络已连接' : '网络未连接' }}</text>
		</view>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				isOnline: false,
				onlineStatus: true
			}
		},
		onLoad() {
			this.checkNetwork()
		},
		methods: {
			checkNetwork() {
				uni.getNetworkType({
					success: (res) => {
						this.isOnline = res.networkType !== 'none'
					}
				})
			},
			startLocalBattle() {
				uni.navigateTo({
					url: '/pages/battle/battle?mode=local'
				})
			},
			startOnlineBattle() {
				if (!this.isOnline) {
					uni.showToast({
						title: '请先连接网络',
						icon: 'none'
					})
					return
				}
				uni.navigateTo({
					url: '/pages/battle/battle?mode=online'
				})
			},
			goToRanking() {
				uni.navigateTo({
					url: '/pages/ranking/ranking'
				})
			},
			goToMine() {
				uni.navigateTo({
					url: '/pages/mine/mine'
				})
			}
		}
	}
</script>

<style scoped>
	.container {
		min-height: 100vh;
		background: linear-gradient(180deg, #8B4513 0%, #DEB887 50%, #F5F5DC 100%);
		padding: 20rpx;
		box-sizing: border-box;
	}
	
	.header {
		display: flex;
		justify-content: space-between;
		align-items: center;
		padding: 60rpx 30rpx 40rpx;
	}
	
	.title-section {
		display: flex;
		flex-direction: column;
	}
	
	.main-title {
		font-size: 48rpx;
		font-weight: bold;
		color: #FFD700;
		text-shadow: 2rpx 2rpx 4rpx rgba(0,0,0,0.3);
	}
	
	.sub-title {
		font-size: 24rpx;
		color: #F5DEB3;
		margin-top: 8rpx;
	}
	
	.logo {
		width: 100rpx;
		height: 100rpx;
		background: radial-gradient(circle, #FFD700 0%, #DAA520 100%);
		border-radius: 50%;
		display: flex;
		align-items: center;
		justify-content: center;
		box-shadow: 0 8rpx 20rpx rgba(0,0,0,0.3);
	}
	
	.logo-text {
		font-size: 48rpx;
		font-weight: bold;
		color: #8B4513;
	}
	
	.menu-section {
		margin-top: 40rpx;
		padding: 0 20rpx;
	}
	
	.menu-item {
		display: flex;
		align-items: center;
		background: rgba(255,255,255,0.95);
		border-radius: 20rpx;
		padding: 30rpx;
		margin-bottom: 20rpx;
		box-shadow: 0 4rpx 15rpx rgba(0,0,0,0.1);
		transition: transform 0.2s, box-shadow 0.2s;
	}
	
	.menu-item:active {
		transform: scale(0.98);
		box-shadow: 0 2rpx 8rpx rgba(0,0,0,0.1);
	}
	
	.menu-icon {
		width: 80rpx;
		height: 80rpx;
		border-radius: 16rpx;
		display: flex;
		align-items: center;
		justify-content: center;
		margin-right: 24rpx;
	}
	
	.local-icon {
		background: linear-gradient(135deg, #4CAF50 0%, #45a049 100%);
	}
	
	.online-icon {
		background: linear-gradient(135deg, #2196F3 0%, #1976D2 100%);
	}
	
	.ranking-icon {
		background: linear-gradient(135deg, #FF9800 0%, #F57C00 100%);
	}
	
	.mine-icon {
		background: linear-gradient(135deg, #9C27B0 0%, #7B1FA2 100%);
	}
	
	.icon-text {
		font-size: 32rpx;
		font-weight: bold;
		color: white;
	}
	
	.menu-info {
		flex: 1;
		display: flex;
		flex-direction: column;
	}
	
	.menu-title {
		font-size: 32rpx;
		font-weight: bold;
		color: #333;
	}
	
	.menu-desc {
		font-size: 24rpx;
		color: #999;
		margin-top: 6rpx;
	}
	
	.menu-arrow {
		font-size: 32rpx;
		color: #ccc;
	}
	
	.online-status {
		position: fixed;
		bottom: 30rpx;
		left: 50%;
		transform: translateX(-50%);
		display: flex;
		align-items: center;
		background: rgba(0,0,0,0.6);
		padding: 12rpx 24rpx;
		border-radius: 30rpx;
	}
	
	.status-dot {
		width: 16rpx;
		height: 16rpx;
		border-radius: 50%;
		background: #999;
		margin-right: 12rpx;
	}
	
	.status-dot.online {
		background: #4CAF50;
		animation: pulse 2s infinite;
	}
	
	@keyframes pulse {
		0%, 100% { opacity: 1; }
		50% { opacity: 0.5; }
	}
	
	.status-text {
		font-size: 24rpx;
		color: white;
	}
</style>