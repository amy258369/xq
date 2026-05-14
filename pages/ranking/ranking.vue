<template>
	<view class="container">
		<view class="ranking-animation" v-if="showAnimation">
			<view class="animation-content">
				<view class="trophy-icon">🏆</view>
				<text class="animation-title">积分榜</text>
				<view class="medal-container">
					<view class="medal gold" :style="{ animationDelay: '0s' }">🥇</view>
					<view class="medal silver" :style="{ animationDelay: '0.2s' }">🥈</view>
					<view class="medal bronze" :style="{ animationDelay: '0.4s' }">🥉</view>
				</view>
			</view>
		</view>
		
		<view class="header">
			<view class="back-btn" @click="goBack">
				<text>← 返回</text>
			</view>
			<text class="header-title">积分榜</text>
			<view class="refresh-btn" @click="refreshRanking">
				<text>刷新</text>
			</view>
		</view>
		
		<view class="ranking-list" v-if="!showAnimation">
			<view class="top-three">
				<view class="rank-item second" v-if="rankings[1]">
					<view class="rank-avatar">
						<text>{{ rankings[1].name.charAt(0) }}</text>
					</view>
					<view class="rank-info">
						<text class="rank-name">{{ rankings[1].name }}</text>
						<text class="rank-score">{{ rankings[1].score }} 分</text>
					</view>
					<view class="rank-badge">2</view>
				</view>
				
				<view class="rank-item first" v-if="rankings[0]">
					<view class="rank-crown">👑</view>
					<view class="rank-avatar">
						<text>{{ rankings[0].name.charAt(0) }}</text>
					</view>
					<view class="rank-info">
						<text class="rank-name">{{ rankings[0].name }}</text>
						<text class="rank-score">{{ rankings[0].score }} 分</text>
					</view>
					<view class="rank-badge">1</view>
				</view>
				
				<view class="rank-item third" v-if="rankings[2]">
					<view class="rank-avatar">
						<text>{{ rankings[2].name.charAt(0) }}</text>
					</view>
					<view class="rank-info">
						<text class="rank-name">{{ rankings[2].name }}</text>
						<text class="rank-score">{{ rankings[2].score }} 分</text>
					</view>
					<view class="rank-badge">3</view>
				</view>
			</view>
			
			<view class="other-ranks">
				<view 
					class="rank-item" 
					v-for="(item, index) in rankings.slice(3)" 
					:key="index"
					:style="{ animationDelay: (index * 0.1) + 's' }"
				>
					<view class="rank-number">{{ index + 4 }}</view>
					<view class="rank-avatar small">
						<text>{{ item.name.charAt(0) }}</text>
					</view>
					<view class="rank-info">
						<text class="rank-name">{{ item.name }}</text>
						<text class="rank-time">用时 {{ formatTime(item.time) }}</text>
					</view>
					<view class="rank-score-value">{{ item.score }} 分</view>
				</view>
			</view>
			
			<view class="empty-state" v-if="rankings.length === 0">
				<text class="empty-icon">📊</text>
				<text class="empty-text">暂无排名数据</text>
				<text class="empty-hint">完成对战后将显示在这里</text>
			</view>
		</view>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				showAnimation: true,
				rankings: []
			}
		},
		onLoad() {
			this.loadRankings()
			this.playAnimation()
		},
		methods: {
			playAnimation() {
				setTimeout(() => {
					this.showAnimation = false
				}, 2000)
			},
			loadRankings() {
				this.rankings = uni.getStorageSync('chess_rankings') || []
				this.rankings.sort((a, b) => b.score - a.score)
			},
			refreshRanking() {
				this.loadRankings()
				uni.showToast({
					title: '已刷新',
					icon: 'success'
				})
			},
			formatTime(seconds) {
				const mins = Math.floor(seconds / 60)
				const secs = seconds % 60
				return `${mins}:${secs.toString().padStart(2, '0')}`
			},
			goBack() {
				uni.navigateBack()
			}
		}
	}
</script>

<style scoped>
	.container {
		min-height: 100vh;
		background: linear-gradient(180deg, #FFD700 0%, #FFA500 30%, #F5DEB3 100%);
	}
	
	.ranking-animation {
		position: fixed;
		top: 0;
		left: 0;
		right: 0;
		bottom: 0;
		background: linear-gradient(180deg, #FFD700 0%, #FFA500 100%);
		display: flex;
		align-items: center;
		justify-content: center;
		z-index: 100;
	}
	
	.animation-content {
		display: flex;
		flex-direction: column;
		align-items: center;
	}
	
	.trophy-icon {
		font-size: 120rpx;
		animation: trophyBounce 1s ease-in-out infinite;
	}
	
	@keyframes trophyBounce {
		0%, 100% { transform: translateY(0); }
		50% { transform: translateY(-30rpx); }
	}
	
	.animation-title {
		font-size: 48rpx;
		font-weight: bold;
		color: #8B4513;
		margin-top: 20rpx;
	}
	
	.medal-container {
		display: flex;
		margin-top: 40rpx;
	}
	
	.medal {
		font-size: 60rpx;
		margin: 0 20rpx;
		opacity: 0;
		animation: medalPop 0.5s ease-out forwards;
	}
	
	@keyframes medalPop {
		0% {
			opacity: 0;
			transform: scale(0);
		}
		50% {
			transform: scale(1.3);
		}
		100% {
			opacity: 1;
			transform: scale(1);
		}
	}
	
	.header {
		display: flex;
		align-items: center;
		justify-content: space-between;
		padding: 60rpx 30rpx 20rpx;
	}
	
	.back-btn, .refresh-btn {
		padding: 15rpx 30rpx;
		background: rgba(255,255,255,0.3);
		border-radius: 30rpx;
	}
	
	.back-btn text, .refresh-btn text {
		color: #8B4513;
		font-size: 28rpx;
		font-weight: bold;
	}
	
	.header-title {
		font-size: 36rpx;
		font-weight: bold;
		color: #8B4513;
	}
	
	.ranking-list {
		padding: 20rpx;
	}
	
	.top-three {
		display: flex;
		justify-content: center;
		align-items: flex-end;
		gap: 20rpx;
		margin-bottom: 40rpx;
	}
	
	.rank-item {
		display: flex;
		flex-direction: column;
		align-items: center;
		padding: 20rpx;
		background: white;
		border-radius: 16rpx;
		box-shadow: 0 4rpx 15rpx rgba(0,0,0,0.1);
		transition: transform 0.3s;
	}
	
	.rank-item.first {
		order: 2;
		padding: 30rpx;
		background: linear-gradient(180deg, #FFD700 0%, #FFEC8B 100%);
	}
	
	.rank-item.second {
		order: 1;
		padding: 25rpx;
		background: linear-gradient(180deg, #C0C0C0 0%, #E8E8E8 100%);
	}
	
	.rank-item.third {
		order: 3;
		padding: 25rpx;
		background: linear-gradient(180deg, #CD7F32 0%, #DAA520 100%);
	}
	
	.rank-crown {
		font-size: 40rpx;
		position: absolute;
		top: -30rpx;
	}
	
	.rank-avatar {
		width: 80rpx;
		height: 80rpx;
		border-radius: 50%;
		background: #8B4513;
		display: flex;
		align-items: center;
		justify-content: center;
		margin-bottom: 10rpx;
		position: relative;
	}
	
	.rank-avatar.small {
		width: 60rpx;
		height: 60rpx;
	}
	
	.rank-avatar text {
		font-size: 32rpx;
		color: white;
		font-weight: bold;
	}
	
	.rank-info {
		display: flex;
		flex-direction: column;
		align-items: center;
	}
	
	.rank-name {
		font-size: 26rpx;
		font-weight: bold;
		color: #333;
	}
	
	.rank-score {
		font-size: 22rpx;
		color: #666;
	}
	
	.rank-time {
		font-size: 20rpx;
		color: #999;
	}
	
	.rank-badge {
		position: absolute;
		top: -10rpx;
		right: -10rpx;
		width: 40rpx;
		height: 40rpx;
		border-radius: 50%;
		background: #FF6B6B;
		display: flex;
		align-items: center;
		justify-content: center;
		font-size: 24rpx;
		font-weight: bold;
		color: white;
	}
	
	.rank-number {
		width: 48rpx;
		height: 48rpx;
		border-radius: 50%;
		background: #f0f0f0;
		display: flex;
		align-items: center;
		justify-content: center;
		font-size: 24rpx;
		font-weight: bold;
		color: #999;
		margin-right: 15rpx;
	}
	
	.other-ranks .rank-item {
		flex-direction: row;
		justify-content: space-between;
		padding: 20rpx;
		margin-bottom: 15rpx;
		opacity: 0;
		animation: fadeInUp 0.5s ease-out forwards;
	}
	
	@keyframes fadeInUp {
		from {
			opacity: 0;
			transform: translateY(20rpx);
		}
		to {
			opacity: 1;
			transform: translateY(0);
		}
	}
	
	.other-ranks .rank-info {
		flex: 1;
		align-items: flex-start;
		margin-left: 15rpx;
	}
	
	.rank-score-value {
		font-size: 28rpx;
		font-weight: bold;
		color: #FF6B6B;
	}
	
	.empty-state {
		display: flex;
		flex-direction: column;
		align-items: center;
		padding: 100rpx 40rpx;
	}
	
	.empty-icon {
		font-size: 80rpx;
		margin-bottom: 20rpx;
	}
	
	.empty-text {
		font-size: 32rpx;
		color: #666;
		margin-bottom: 10rpx;
	}
	
	.empty-hint {
		font-size: 24rpx;
		color: #999;
	}
</style>