<template>
	<view class="container">
		<view class="header">
			<view class="back-btn" @click="goBack">
				<text>← 返回</text>
			</view>
			<text class="header-title">我的</text>
			<view class="settings-btn" @click="showSettings">
				<text>⚙️</text>
			</view>
		</view>
		
		<view class="profile-section">
			<view class="avatar-container">
				<view class="avatar">
					<text>{{ userName.charAt(0) }}</text>
				</view>
				<view class="level-badge">Lv.{{ userLevel }}</view>
			</view>
			<view class="profile-info">
				<text class="user-name">{{ userName }}</text>
				<text class="user-title">棋士</text>
			</view>
		</view>
		
		<view class="stats-section">
			<view class="stat-item">
				<text class="stat-value">{{ stats.wins }}</text>
				<text class="stat-label">胜场</text>
			</view>
			<view class="stat-divider"></view>
			<view class="stat-item">
				<text class="stat-value">{{ stats.losses }}</text>
				<text class="stat-label">负场</text>
			</view>
			<view class="stat-divider"></view>
			<view class="stat-item">
				<text class="stat-value">{{ stats.draws }}</text>
				<text class="stat-label">平局</text>
			</view>
			<view class="stat-divider"></view>
			<view class="stat-item">
				<text class="stat-value">{{ winRate }}%</text>
				<text class="stat-label">胜率</text>
			</view>
		</view>
		
		<view class="menu-section">
			<view class="menu-item" @click="viewHistory">
				<view class="menu-icon">📜</view>
				<view class="menu-info">
					<text class="menu-title">对战记录</text>
					<text class="menu-desc">查看历史对局</text>
				</view>
				<view class="menu-arrow">→</view>
			</view>
			
			<view class="menu-item" @click="viewAchievements">
				<view class="menu-icon">🏅</view>
				<view class="menu-info">
					<text class="menu-title">成就</text>
					<text class="menu-desc">查看获得的成就</text>
				</view>
				<view class="menu-arrow">→</view>
			</view>
			
			<view class="menu-item" @click="editProfile">
				<view class="menu-icon">✏️</view>
				<view class="menu-info">
					<text class="menu-title">编辑资料</text>
					<text class="menu-desc">修改昵称等信息</text>
				</view>
				<view class="menu-arrow">→</view>
			</view>
			
			<view class="menu-item" @click="shareApp">
				<view class="menu-icon">📤</view>
				<view class="menu-info">
					<text class="menu-title">分享应用</text>
					<text class="menu-desc">邀请好友一起玩</text>
				</view>
				<view class="menu-arrow">→</view>
			</view>
			
			<view class="menu-item" @click="aboutApp">
				<view class="menu-icon">ℹ️</view>
				<view class="menu-info">
					<text class="menu-title">关于我们</text>
					<text class="menu-desc">版本信息与反馈</text>
				</view>
				<view class="menu-arrow">→</view>
			</view>
		</view>
		
		<view class="quick-actions">
			<view class="action-item" @click="startQuickGame">
				<view class="action-icon">🎮</view>
				<text class="action-text">快速开始</text>
			</view>
			<view class="action-item" @click="goToRanking">
				<view class="action-icon">🏆</view>
				<text class="action-text">排行榜</text>
			</view>
		</view>
		
		<view class="edit-modal" v-if="showEditModal">
			<view class="modal-content">
				<text class="modal-title">修改昵称</text>
				<input 
					class="nickname-input" 
					v-model="newNickname" 
					placeholder="请输入新昵称"
					maxlength="10"
				/>
				<view class="modal-btns">
					<view class="modal-btn cancel" @click="closeEditModal">
						<text>取消</text>
					</view>
					<view class="modal-btn confirm" @click="saveNickname">
						<text>保存</text>
					</view>
				</view>
			</view>
		</view>
		
		<view class="about-modal" v-if="showAboutModal">
			<view class="modal-content">
				<text class="modal-title">关于天天象棋</text>
				<view class="about-info">
					<text class="version">版本 1.0.0</text>
					<text class="description">一款经典的中国象棋游戏，支持双人对战和联机对战。</text>
					<text class="copyright">© 2024 天天象棋</text>
				</view>
				<view class="modal-btns">
					<view class="modal-btn confirm" @click="closeAboutModal">
						<text>确定</text>
					</view>
				</view>
			</view>
		</view>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				userName: '玩家一',
				userLevel: 1,
				stats: {
					wins: 0,
					losses: 0,
					draws: 0
				},
				newNickname: '',
				showEditModal: false,
				showAboutModal: false
			}
		},
		computed: {
			winRate() {
				const total = this.stats.wins + this.stats.losses + this.stats.draws
				if (total === 0) return 0
				return Math.round((this.stats.wins / total) * 100)
			}
		},
		onLoad() {
			this.loadUserData()
		},
		methods: {
			loadUserData() {
				const userData = uni.getStorageSync('chess_user') || {}
				this.userName = userData.name || '玩家一'
				this.userLevel = userData.level || 1
				this.stats = userData.stats || { wins: 0, losses: 0, draws: 0 }
			},
			saveUserData() {
				uni.setStorageSync('chess_user', {
					name: this.userName,
					level: this.userLevel,
					stats: this.stats
				})
			},
			goBack() {
				uni.navigateBack()
			},
			showSettings() {
				uni.showToast({
					title: '设置功能开发中',
					icon: 'none'
				})
			},
			viewHistory() {
				uni.showToast({
					title: '对战记录开发中',
					icon: 'none'
				})
			},
			viewAchievements() {
				uni.showToast({
					title: '成就系统开发中',
					icon: 'none'
				})
			},
			editProfile() {
				this.newNickname = this.userName
				this.showEditModal = true
			},
			closeEditModal() {
				this.showEditModal = false
			},
			saveNickname() {
				if (this.newNickname.trim()) {
					this.userName = this.newNickname.trim()
					this.saveUserData()
					uni.showToast({
						title: '修改成功',
						icon: 'success'
					})
				} else {
					uni.showToast({
						title: '请输入昵称',
						icon: 'none'
					})
				}
				this.showEditModal = false
			},
			shareApp() {
				uni.showShareMenu({
					withShareTicket: true
				})
			},
			aboutApp() {
				this.showAboutModal = true
			},
			closeAboutModal() {
				this.showAboutModal = false
			},
			startQuickGame() {
				uni.navigateTo({
					url: '/pages/battle/battle?mode=local'
				})
			},
			goToRanking() {
				uni.navigateTo({
					url: '/pages/ranking/ranking'
				})
			}
		}
	}
</script>

<style scoped>
	.container {
		min-height: 100vh;
		background: linear-gradient(180deg, #9C27B0 0%, #673AB7 30%, #EDE7F6 100%);
	}
	
	.header {
		display: flex;
		align-items: center;
		justify-content: space-between;
		padding: 60rpx 30rpx 20rpx;
	}
	
	.back-btn {
		padding: 15rpx 30rpx;
		background: rgba(255,255,255,0.2);
		border-radius: 30rpx;
	}
	
	.back-btn text {
		color: white;
		font-size: 28rpx;
	}
	
	.header-title {
		font-size: 36rpx;
		font-weight: bold;
		color: white;
	}
	
	.settings-btn text {
		font-size: 36rpx;
	}
	
	.profile-section {
		display: flex;
		align-items: center;
		padding: 40rpx 30rpx;
	}
	
	.avatar-container {
		position: relative;
		margin-right: 24rpx;
	}
	
	.avatar {
		width: 120rpx;
		height: 120rpx;
		border-radius: 50%;
		background: linear-gradient(135deg, #FFD700 0%, #FFA500 100%);
		display: flex;
		align-items: center;
		justify-content: center;
		box-shadow: 0 8rpx 20rpx rgba(0,0,0,0.2);
	}
	
	.avatar text {
		font-size: 48rpx;
		font-weight: bold;
		color: #8B4513;
	}
	
	.level-badge {
		position: absolute;
		bottom: -5rpx;
		left: 50%;
		transform: translateX(-50%);
		background: #FF6B6B;
		padding: 5rpx 15rpx;
		border-radius: 20rpx;
		font-size: 20rpx;
		font-weight: bold;
		color: white;
	}
	
	.profile-info {
		display: flex;
		flex-direction: column;
	}
	
	.user-name {
		font-size: 36rpx;
		font-weight: bold;
		color: white;
	}
	
	.user-title {
		font-size: 24rpx;
		color: rgba(255,255,255,0.8);
		margin-top: 8rpx;
	}
	
	.stats-section {
		display: flex;
		align-items: center;
		justify-content: space-around;
		background: rgba(255,255,255,0.95);
		margin: 0 30rpx;
		border-radius: 20rpx;
		padding: 30rpx;
		box-shadow: 0 4rpx 15rpx rgba(0,0,0,0.1);
	}
	
	.stat-item {
		display: flex;
		flex-direction: column;
		align-items: center;
	}
	
	.stat-value {
		font-size: 40rpx;
		font-weight: bold;
		color: #673AB7;
	}
	
	.stat-label {
		font-size: 24rpx;
		color: #999;
		margin-top: 8rpx;
	}
	
	.stat-divider {
		width: 1rpx;
		height: 60rpx;
		background: #eee;
	}
	
	.menu-section {
		margin-top: 30rpx;
		padding: 0 30rpx;
	}
	
	.menu-item {
		display: flex;
		align-items: center;
		background: rgba(255,255,255,0.95);
		border-radius: 16rpx;
		padding: 25rpx;
		margin-bottom: 15rpx;
		box-shadow: 0 4rpx 15rpx rgba(0,0,0,0.1);
	}
	
	.menu-icon {
		font-size: 40rpx;
		margin-right: 20rpx;
	}
	
	.menu-info {
		flex: 1;
		display: flex;
		flex-direction: column;
	}
	
	.menu-title {
		font-size: 30rpx;
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
	
	.quick-actions {
		display: flex;
		gap: 20rpx;
		padding: 30rpx;
	}
	
	.action-item {
		flex: 1;
		display: flex;
		flex-direction: column;
		align-items: center;
		background: rgba(255,255,255,0.95);
		border-radius: 16rpx;
		padding: 30rpx;
		box-shadow: 0 4rpx 15rpx rgba(0,0,0,0.1);
	}
	
	.action-icon {
		font-size: 48rpx;
		margin-bottom: 10rpx;
	}
	
	.action-text {
		font-size: 28rpx;
		font-weight: bold;
		color: #673AB7;
	}
	
	.edit-modal, .about-modal {
		position: fixed;
		top: 0;
		left: 0;
		right: 0;
		bottom: 0;
		background: rgba(0,0,0,0.7);
		display: flex;
		align-items: center;
		justify-content: center;
		z-index: 100;
	}
	
	.modal-content {
		background: white;
		border-radius: 20rpx;
		padding: 40rpx;
		width: 80%;
	}
	
	.modal-title {
		font-size: 36rpx;
		font-weight: bold;
		color: #333;
		text-align: center;
		margin-bottom: 30rpx;
	}
	
	.nickname-input {
		width: 100%;
		height: 80rpx;
		border: 2rpx solid #eee;
		border-radius: 10rpx;
		padding: 0 20rpx;
		font-size: 30rpx;
		box-sizing: border-box;
		margin-bottom: 30rpx;
	}
	
	.modal-btns {
		display: flex;
		gap: 20rpx;
	}
	
	.modal-btn {
		flex: 1;
		padding: 25rpx;
		border-radius: 10rpx;
		text-align: center;
	}
	
	.modal-btn.cancel {
		background: #f0f0f0;
	}
	
	.modal-btn.cancel text {
		color: #666;
		font-size: 30rpx;
	}
	
	.modal-btn.confirm {
		background: #673AB7;
	}
	
	.modal-btn.confirm text {
		color: white;
		font-size: 30rpx;
		font-weight: bold;
	}
	
	.about-info {
		display: flex;
		flex-direction: column;
		align-items: center;
	}
	
	.version {
		font-size: 28rpx;
		color: #666;
		margin-bottom: 15rpx;
	}
	
	.description {
		font-size: 26rpx;
		color: #999;
		text-align: center;
		margin-bottom: 15rpx;
	}
	
	.copyright {
		font-size: 24rpx;
		color: #bbb;
	}
</style>