<template>
	<view class="container">
		<view class="battle-animation" v-if="showAnimation">
			<view class="animation-content">
				<view class="pk-text">
					<text class="pk-char">楚</text>
					<text class="pk-vs">VS</text>
					<text class="pk-char">汉</text>
				</view>
				<view class="animation-pieces">
					<view class="piece black" :style="{ animationDelay: '0.1s' }">兵</view>
					<view class="piece black" :style="{ animationDelay: '0.2s' }">馬</view>
					<view class="piece black" :style="{ animationDelay: '0.3s' }">象</view>
					<view class="piece black" :style="{ animationDelay: '0.4s' }">将</view>
					<view class="piece black" :style="{ animationDelay: '0.5s' }">車</view>
					<view class="piece red" :style="{ animationDelay: '0.6s' }">兵</view>
					<view class="piece red" :style="{ animationDelay: '0.7s' }">馬</view>
					<view class="piece red" :style="{ animationDelay: '0.8s' }">相</view>
					<view class="piece red" :style="{ animationDelay: '0.9s' }">帅</view>
					<view class="piece red" :style="{ animationDelay: '1.0s' }">車</view>
				</view>
			</view>
		</view>
		
		<view class="battle-header">
			<view class="back-btn" @click="goBack">
				<text>← 返回</text>
			</view>
			<view class="battle-info">
				<text class="battle-mode">{{ battleMode }}</text>
			</view>
			<view class="timer" v-if="gameStarted">
				<text>{{ formatTime(timer) }}</text>
			</view>
		</view>
		
		<view class="players-bar">
			<view class="player red-player" :class="{ active: currentPlayer === 'red' && gameStarted }">
				<view class="player-avatar">
					<text>{{ onlineMode ? opponentName : '玩家二' }}</text>
				</view>
				<view class="player-info">
					<text class="player-name">{{ onlineMode ? opponentName : '红方' }}</text>
					<view class="captured-pieces">
						<text v-for="(piece, idx) in capturedRed" :key="idx" class="captured">{{ piece }}</text>
					</view>
				</view>
				<view class="player-turn" v-if="currentPlayer === 'red' && gameStarted">
					<text>执子中</text>
				</view>
			</view>
			
			<view class="player black-player" :class="{ active: currentPlayer === 'black' && gameStarted }">
				<view class="player-avatar">
					<text>{{ '玩家一' }}</text>
				</view>
				<view class="player-info">
					<text class="player-name">黑方</text>
					<view class="captured-pieces">
						<text v-for="(piece, idx) in capturedBlack" :key="idx" class="captured">{{ piece }}</text>
					</view>
				</view>
				<view class="player-turn" v-if="currentPlayer === 'black' && gameStarted">
					<text>执子中</text>
				</view>
			</view>
		</view>
		
		<view class="board-container">
			<view class="board" @click="handleBoardClick">
				<view class="board-grid">
					<view class="horizontal-lines">
						<view v-for="i in 10" :key="'h'+i" class="h-line" :style="{ top: ((i-1)*52) + 'rpx' }"></view>
					</view>
					<view class="vertical-lines">
						<view v-for="i in 9" :key="'v'+i" class="v-line" :style="{ left: ((i-1)*52) + 'rpx' }"></view>
					</view>
					<view class="river-line"></view>
					<view class="palace-lines">
						<view class="palace top-palace"></view>
						<view class="palace bottom-palace"></view>
					</view>
					<view class="star-points">
						<view v-for="(star, idx) in starPoints" :key="idx" 
							class="star" 
							:style="{ left: star.x + 'rpx', top: star.y + 'rpx' }"
						></view>
					</view>
				</view>
				<view class="board-overlay">
					<view 
						v-for="(row, y) in board" 
						:key="y" 
					>
						<view 
							v-for="(cell, x) in row" 
							:key="x" 
							class="board-cell"
							:class="{ 
								'selected': selectedPos && selectedPos.x === x && selectedPos.y === y,
								'highlight': isValidMove(x, y)
							}"
							:style="{ left: (x * 52) + 'rpx', top: (y * 52) + 'rpx' }"
							@click.stop="handleCellClick(x, y)"
						>
							<view 
								v-if="cell" 
								class="piece"
								:class="[cell.color, { 'selected': selectedPos && selectedPos.x === x && selectedPos.y === y }]"
							>
								<text class="piece-text">{{ cell.type }}</text>
							</view>
						</view>
					</view>
				</view>
				<view class="river-text">
					<text>楚河</text>
					<text>汉界</text>
				</view>
			</view>
		</view>
		
		<view class="action-bar" v-if="gameStarted">
			<view class="action-btn" @click="undoMove">
				<text>悔棋</text>
			</view>
			<view class="action-btn" @click="restartGame">
				<text>重新开始</text>
			</view>
			<view class="action-btn" @click="resign">
				<text>认输</text>
			</view>
		</view>
		
		<view class="online-lobby" v-if="onlineMode && !gameStarted">
			<view class="lobby-title">
				<text>等待对手加入...</text>
			</view>
			<view class="room-code">
				<text class="code-label">房间码</text>
				<text class="code-value">{{ roomCode }}</text>
			</view>
			<view class="copy-btn" @click="copyRoomCode">
				<text>复制房间码</text>
			</view>
			<view class="cancel-btn" @click="cancelLobby">
				<text>取消</text>
			</view>
		</view>
		
		<view class="game-over-modal" v-if="gameOver">
			<view class="modal-content">
				<text class="result-text">{{ winner }} 获胜!</text>
				<view class="modal-btns">
					<view class="modal-btn" @click="restartGame">
						<text>再来一局</text>
					</view>
					<view class="modal-btn secondary" @click="goBack">
						<text>返回首页</text>
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
				showAnimation: true,
				gameStarted: false,
				gameOver: false,
				winner: '',
				currentPlayer: 'red',
				timer: 0,
				timerInterval: null,
				selectedPos: null,
				board: [],
				capturedRed: [],
				capturedBlack: [],
				moveHistory: [],
				onlineMode: false,
				roomCode: '',
				opponentName: '',
				isHost: false,
				starPoints: [
					{ x: 104, y: 0 }, { x: 312, y: 0 },
					{ x: 208, y: 156 }, { x: 208, y: 260 },
					{ x: 104, y: 468 }, { x: 312, y: 468 }
				]
			}
		},
		computed: {
			battleMode() {
				return this.onlineMode ? '联机对战' : '双人对战'
			}
		},
		onLoad(options) {
			this.onlineMode = options?.mode === 'online'
			this.initBoard()
			this.playAnimation()
		},
		onUnload() {
			if (this.timerInterval) {
				clearInterval(this.timerInterval)
			}
			if (this.socket) {
				this.socket.close()
			}
		},
		methods: {
			playAnimation() {
				setTimeout(() => {
					this.showAnimation = false
					if (this.onlineMode) {
						this.createRoom()
					} else {
						this.startGame()
					}
				}, 2500)
			},
			initBoard() {
				this.board = Array(10).fill(null).map(() => Array(9).fill(null))
				this.initPieces()
			},
			initPieces() {
				const pieces = {
					black: [
						{ type: '車', x: 0, y: 0 }, { type: '馬', x: 1, y: 0 },
						{ type: '象', x: 2, y: 0 }, { type: '仕', x: 3, y: 0 },
						{ type: '将', x: 4, y: 0 }, { type: '仕', x: 5, y: 0 },
						{ type: '象', x: 6, y: 0 }, { type: '馬', x: 7, y: 0 },
						{ type: '車', x: 8, y: 0 }, { type: '炮', x: 1, y: 2 },
						{ type: '炮', x: 7, y: 2 }, { type: '卒', x: 0, y: 3 },
						{ type: '卒', x: 2, y: 3 }, { type: '卒', x: 4, y: 3 },
						{ type: '卒', x: 6, y: 3 }, { type: '卒', x: 8, y: 3 }
					],
					red: [
						{ type: '車', x: 0, y: 9 }, { type: '馬', x: 1, y: 9 },
						{ type: '相', x: 2, y: 9 }, { type: '仕', x: 3, y: 9 },
						{ type: '帅', x: 4, y: 9 }, { type: '仕', x: 5, y: 9 },
						{ type: '相', x: 6, y: 9 }, { type: '馬', x: 7, y: 9 },
						{ type: '車', x: 8, y: 9 }, { type: '炮', x: 1, y: 7 },
						{ type: '炮', x: 7, y: 7 }, { type: '兵', x: 0, y: 6 },
						{ type: '兵', x: 2, y: 6 }, { type: '兵', x: 4, y: 6 },
						{ type: '兵', x: 6, y: 6 }, { type: '兵', x: 8, y: 6 }
					]
				}
				pieces.black.forEach(p => {
					this.board[p.y][p.x] = { type: p.type, color: 'black' }
				})
				pieces.red.forEach(p => {
					this.board[p.y][p.x] = { type: p.type, color: 'red' }
				})
			},
			startGame() {
				this.gameStarted = true
				this.startTimer()
			},
			startTimer() {
				this.timerInterval = setInterval(() => {
					this.timer++
				}, 1000)
			},
			formatTime(seconds) {
				const mins = Math.floor(seconds / 60)
				const secs = seconds % 60
				return `${mins.toString().padStart(2, '0')}:${secs.toString().padStart(2, '0')}`
			},
			createRoom() {
				this.roomCode = Math.random().toString(36).substr(2, 6).toUpperCase()
				this.isHost = true
			},
			copyRoomCode() {
				uni.setClipboardData({
					data: this.roomCode,
					success: () => {
						uni.showToast({ title: '已复制', icon: 'success' })
					}
				})
			},
			cancelLobby() {
				uni.navigateBack()
			},
			handleBoardClick() {
				if (this.selectedPos) {
					this.selectedPos = null
				}
			},
			handleCellClick(x, y) {
				if (!this.gameStarted || this.gameOver) return
				
				const piece = this.board[y][x]
				
				if (this.selectedPos) {
					if (this.selectedPos.x === x && this.selectedPos.y === y) {
						this.selectedPos = null
						return
					}
					
					if (this.isValidMove(x, y)) {
						this.makeMove(this.selectedPos.x, this.selectedPos.y, x, y)
						this.selectedPos = null
						return
					}
				}
				
				if (piece && piece.color === this.currentPlayer) {
					this.selectedPos = { x, y }
				}
			},
			isValidMove(targetX, targetY) {
				if (!this.selectedPos) return false
				
				const { x, y } = this.selectedPos
				const piece = this.board[y][x]
				const targetPiece = this.board[targetY][targetX]
				
				if (targetPiece && targetPiece.color === piece.color) {
					return false
				}
				
				const dx = Math.abs(targetX - x)
				const absDy = Math.abs(targetY - y)
				
				switch (piece.type) {
					case '車':
						return this.isStraightMove(x, y, targetX, targetY)
					case '馬':
						if (!this.isHorseMoveValid(x, y, targetX, targetY)) return false
						return (dx === 2 && absDy === 1) || (dx === 1 && absDy === 2)
					case '象':
						if (!this.isInOwnCamp(targetY, piece.color)) return false
						if (dx !== 2 || absDy !== 2) return false
						return !this.board[Math.floor((y + targetY)/2)][Math.floor((x + targetX)/2)]
					case '相':
						if (!this.isInOwnCamp(targetY, piece.color)) return false
						if (dx !== 2 || absDy !== 2) return false
						return !this.board[Math.floor((y + targetY)/2)][Math.floor((x + targetX)/2)]
					case '仕':
						if (!this.isInPalace(targetX, targetY, piece.color)) return false
						return dx === 1 && absDy === 1
					case '将':
						if (!this.isInPalace(targetX, targetY, piece.color)) return false
						return (dx === 1 && absDy === 0) || (dx === 0 && absDy === 1)
					case '帅':
						if (!this.isInPalace(targetX, targetY, piece.color)) return false
						return (dx === 1 && absDy === 0) || (dx === 0 && absDy === 1)
					case '炮':
						if (!targetPiece) {
							return this.isStraightMove(x, y, targetX, targetY)
						} else {
							return this.isCannonMove(x, y, targetX, targetY)
						}
					case '卒':
					case '兵':
						return this.isPawnMoveValid(x, y, targetX, targetY, piece.color, targetPiece)
					default:
						return false
				}
			},
			isStraightMove(x1, y1, x2, y2) {
				if (x1 !== x2 && y1 !== y2) return false
				
				const minX = Math.min(x1, x2)
				const maxX = Math.max(x1, x2)
				const minY = Math.min(y1, y2)
				const maxY = Math.max(y1, y2)
				
				for (let x = minX + 1; x < maxX; x++) {
					if (this.board[y1][x]) return false
				}
				for (let y = minY + 1; y < maxY; y++) {
					if (this.board[y][x1]) return false
				}
				return true
			},
			isCannonMove(x1, y1, x2, y2) {
				if (x1 !== x2 && y1 !== y2) return false
				
				let count = 0
				const minX = Math.min(x1, x2)
				const maxX = Math.max(x1, x2)
				const minY = Math.min(y1, y2)
				const maxY = Math.max(y1, y2)
				
				for (let x = minX + 1; x < maxX; x++) {
					if (this.board[y1][x]) count++
				}
				for (let y = minY + 1; y < maxY; y++) {
					if (this.board[y][x1]) count++
				}
				return count === 1
			},
			isHorseMoveValid(x, y, targetX, targetY) {
				const dx = Math.abs(targetX - x)
				const dy = Math.abs(targetY - y)
				
				let blockX = x
				let blockY = y
				
				if (dx === 2 && dy === 1) {
					blockX = x + (targetX > x ? 1 : -1)
					blockY = y
				} else if (dx === 1 && dy === 2) {
					blockX = x
					blockY = y + (targetY > y ? 1 : -1)
				}
				
				return !this.board[blockY] || !this.board[blockY][blockX]
			},
			isPawnMoveValid(x, y, targetX, targetY, color, targetPiece) {
				const dx = Math.abs(targetX - x)
				const dy = targetY - y
				
				if (color === 'black') {
					if (dy === 1 && dx === 0) {
						return true
					}
					if (y >= 5 && dx === 1 && dy === 0) {
						return true
					}
					return false
				} else {
					if (dy === -1 && dx === 0) {
						return true
					}
					if (y <= 4 && dx === 1 && dy === 0) {
						return true
					}
					return false
				}
			},
			isInOwnCamp(y, color) {
				return color === 'black' ? y <= 4 : y >= 5
			},
			isInEnemyCamp(y, color) {
				return color === 'black' ? y >= 5 : y <= 4
			},
			isInPalace(x, y, color) {
				if (x < 3 || x > 5) return false
				if (color === 'black') {
					return y >= 0 && y <= 2
				} else {
					return y >= 7 && y <= 9
				}
			},
			makeMove(fromX, fromY, toX, toY) {
				const piece = this.board[fromY][fromX]
				const captured = this.board[toY][toX]
				
				this.moveHistory.push({
					from: { x: fromX, y: fromY },
					to: { x: toX, y: toY },
					piece: piece.type,
					captured: captured ? captured.type : null
				})
				
				this.board[toY][toX] = piece
				this.board[fromY][fromX] = null
				
				if (captured) {
					if (captured.color === 'red') {
						this.capturedRed.push(captured.type)
					} else {
						this.capturedBlack.push(captured.type)
					}
					this.checkGameOver(captured.type)
				}
				this.switchPlayer()
			},
			switchPlayer() {
				this.currentPlayer = this.currentPlayer === 'black' ? 'red' : 'black'
			},
			checkGameOver(capturedType) {
				if (capturedType === '帅' || capturedType === '将') {
					this.gameOver = true
					this.winner = this.currentPlayer === 'black' ? '红方' : '黑方'
					this.stopTimer()
					this.updateRanking()
				}
			},
			stopTimer() {
				if (this.timerInterval) {
					clearInterval(this.timerInterval)
					this.timerInterval = null
				}
			},
			updateRanking() {
				const rankings = uni.getStorageSync('chess_rankings') || []
				const winnerData = {
					name: this.currentPlayer === 'black' ? '玩家一' : (this.onlineMode ? this.opponentName : '玩家二'),
					score: 100,
					time: this.timer,
					date: new Date().toISOString()
				}
				rankings.push(winnerData)
				rankings.sort((a, b) => b.score - a.score)
				uni.setStorageSync('chess_rankings', rankings.slice(0, 100))
			},
			undoMove() {
				if (this.moveHistory.length === 0) return
				
				const lastMove = this.moveHistory.pop()
				const piece = this.board[lastMove.to.y][lastMove.to.x]
				
				this.board[lastMove.from.y][lastMove.from.x] = piece
				this.board[lastMove.to.y][lastMove.to.x] = lastMove.captured ? 
					{ type: lastMove.captured, color: this.currentPlayer } : null
				
				if (lastMove.captured) {
					if (this.currentPlayer === 'red') {
						this.capturedRed.pop()
					} else {
						this.capturedBlack.pop()
					}
				}
				
				this.switchPlayer()
			},
			restartGame() {
				this.gameOver = false
				this.winner = ''
				this.currentPlayer = 'black'
				this.timer = 0
				this.selectedPos = null
				this.capturedRed = []
				this.capturedBlack = []
				this.moveHistory = []
				this.initBoard()
				this.startGame()
			},
			resign() {
				uni.showModal({
					title: '确认认输',
					content: '确定要认输吗？',
					success: (res) => {
						if (res.confirm) {
							this.gameOver = true
							this.winner = this.currentPlayer === 'black' ? '红方' : '黑方'
							this.stopTimer()
							this.updateRanking()
						}
					}
				})
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
		background: linear-gradient(180deg, #2C5F2D 0%, #97BC62 100%);
		position: relative;
	}
	
	.battle-animation {
		position: fixed;
		top: 0;
		left: 0;
		right: 0;
		bottom: 0;
		background: linear-gradient(180deg, #1a1a2e 0%, #16213e 100%);
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
	
	.pk-text {
		display: flex;
		align-items: center;
		margin-bottom: 60rpx;
	}
	
	.pk-char {
		font-size: 120rpx;
		font-weight: bold;
		color: #FFD700;
		animation: pkPulse 0.5s ease-in-out infinite;
	}
	
	.pk-vs {
		font-size: 60rpx;
		font-weight: bold;
		color: #FF4444;
		margin: 0 30rpx;
		animation: vsShake 0.5s ease-in-out infinite;
	}
	
	@keyframes pkPulse {
		0%, 100% { transform: scale(1); }
		50% { transform: scale(1.2); }
	}
	
	@keyframes vsShake {
		0%, 100% { transform: scale(1) rotate(0deg); }
		25% { transform: scale(1.1) rotate(-5deg); }
		75% { transform: scale(1.1) rotate(5deg); }
	}
	
	.animation-pieces {
		display: flex;
		width: 600rpx;
		justify-content: space-between;
	}
	
	.animation-pieces .piece {
		width: 60rpx;
		height: 60rpx;
		border-radius: 50%;
		display: flex;
		align-items: center;
		justify-content: center;
		font-size: 32rpx;
		font-weight: bold;
		animation: pieceFly 1s ease-out forwards;
		opacity: 0;
	}
	
	.animation-pieces .black {
		background: linear-gradient(135deg, #444 0%, #222 100%);
		color: #FFD700;
		border: 2rpx solid #666;
	}
	
	.animation-pieces .red {
		background: linear-gradient(135deg, #DC143C 0%, #8B0000 100%);
		color: white;
		border: 2rpx solid #FF6B6B;
	}
	
	@keyframes pieceFly {
		0% {
			opacity: 0;
			transform: translateY(-100rpx) scale(0);
		}
		50% {
			opacity: 1;
			transform: translateY(0) scale(1.2);
		}
		100% {
			opacity: 1;
			transform: translateY(0) scale(1);
		}
	}
	
	.battle-header {
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
	
	.battle-info .battle-mode {
		font-size: 32rpx;
		font-weight: bold;
		color: #FFD700;
	}
	
	.timer text {
		font-size: 28rpx;
		color: white;
		font-family: monospace;
	}
	
	.players-bar {
		display: flex;
		justify-content: space-between;
		padding: 0 30rpx;
		margin-bottom: 20rpx;
	}
	
	.player {
		display: flex;
		align-items: center;
		background: rgba(255,255,255,0.9);
		border-radius: 16rpx;
		padding: 15rpx 20rpx;
		min-width: 200rpx;
		transition: all 0.3s;
	}
	
	.player.active {
		border: 3rpx solid #FFD700;
		box-shadow: 0 0 20rpx rgba(255,215,0,0.5);
	}
	
	.player-avatar {
		width: 60rpx;
		height: 60rpx;
		border-radius: 50%;
		display: flex;
		align-items: center;
		justify-content: center;
		margin-right: 15rpx;
	}
	
	.black-player .player-avatar {
		background: linear-gradient(135deg, #333 0%, #111 100%);
	}
	
	.black-player .player-avatar text {
		color: #FFD700;
		font-size: 24rpx;
		font-weight: bold;
	}
	
	.red-player .player-avatar {
		background: linear-gradient(135deg, #DC143C 0%, #8B0000 100%);
	}
	
	.red-player .player-avatar text {
		color: white;
		font-size: 24rpx;
		font-weight: bold;
	}
	
	.player-info {
		display: flex;
		flex-direction: column;
	}
	
	.player-name {
		font-size: 26rpx;
		font-weight: bold;
		color: #333;
	}
	
	.captured-pieces {
		display: flex;
		flex-wrap: wrap;
		max-width: 120rpx;
	}
	
	.captured {
		font-size: 20rpx;
		color: #999;
		margin-right: 4rpx;
	}
	
	.player-turn {
		margin-left: 10rpx;
		background: #FFD700;
		padding: 5rpx 10rpx;
		border-radius: 10rpx;
	}
	
	.player-turn text {
		font-size: 20rpx;
		color: #8B4513;
		font-weight: bold;
	}
	
	.board-container {
		display: flex;
		justify-content: center;
		padding: 20rpx;
	}
	
	.board {
		width: 460rpx;
		height: 520rpx;
		background: linear-gradient(135deg, #DEB887 0%, #D2B48C 100%);
		border-radius: 8rpx;
		position: relative;
		box-shadow: 0 10rpx 30rpx rgba(0,0,0,0.4);
		border: 4rpx solid #8B4513;
	}
	
	.board-grid {
		position: absolute;
		top: 0;
		left: 0;
		width: 100%;
		height: 100%;
		overflow: hidden;
	}
	
	.horizontal-lines .h-line {
		position: absolute;
		left: 0;
		right: 0;
		height: 2rpx;
		background: #333;
	}
	
	.vertical-lines .v-line {
		position: absolute;
		top: 0;
		bottom: 0;
		width: 2rpx;
		background: #333;
	}
	
	.vertical-lines .v-line:nth-child(1) {
		left: 0 !important;
	}
	.vertical-lines .v-line:nth-child(9) {
		left: 416rpx !important;
	}
	
	.vertical-lines .v-line:nth-child(5) {
		height: 208rpx;
		top: 0;
	}

	.vertical-lines .v-line:nth-child(5)::after {
		content: '';
		position: absolute;
		bottom: 0;
		left: 0;
		width: 100%;
		height: 208rpx;
		background: #333;
		margin-bottom: -52rpx;
	}
	
	.river-line {
		position: absolute;
		top: 208rpx;
		left: 0;
		right: 0;
		height: 104rpx;
		background: linear-gradient(90deg, 
			transparent 0%, 
			rgba(139, 69, 19, 0.3) 10%, 
			rgba(139, 69, 19, 0.5) 50%, 
			rgba(139, 69, 19, 0.3) 90%, 
			transparent 100%
		);
	}
	
	.palace-lines {
		position: absolute;
		top: 0;
		left: 0;
		width: 100%;
		height: 100%;
	}
	
	.palace {
		position: absolute;
		width: 156rpx;
		height: 156rpx;
		border: 2rpx solid #333;
	}

	.top-palace {
		top: 0;
		left: 156rpx;
		border-bottom: none;
	}

	.bottom-palace {
		bottom: 0;
		left: 156rpx;
		border-top: none;
	}

	.top-palace::before,
	.bottom-palace::before,
	.top-palace::after,
	.bottom-palace::after {
		content: '';
		position: absolute;
		width: 100%;
		height: 2rpx;
		background: #333;
	}

	.top-palace::before {
		top: 50%;
		left: 0;
		transform: rotate(45deg);
		transform-origin: left center;
	}

	.top-palace::after {
		top: 50%;
		right: 0;
		transform: rotate(-45deg);
		transform-origin: right center;
	}

	.bottom-palace::before {
		top: 50%;
		left: 0;
		transform: rotate(-45deg);
		transform-origin: left center;
	}

	.bottom-palace::after {
		top: 50%;
		right: 0;
		transform: rotate(45deg);
		transform-origin: right center;
	}
	
	.top-palace::before,
	.bottom-palace::before,
	.top-palace::after,
	.bottom-palace::after {
		content: '';
		position: absolute;
		width: 100%;
		height: 2rpx;
		background: #333;
		transform: rotate(45deg);
	}
	
	.top-palace::before {
		top: 50%;
		left: 0;
		transform-origin: left center;
	}
	
	.top-palace::after {
		top: 50%;
		right: 0;
		transform-origin: right center;
		transform: rotate(-45deg);
	}
	
	.bottom-palace::before {
		top: 50%;
		left: 0;
		transform-origin: left center;
	}
	
	.bottom-palace::after {
		top: 50%;
		right: 0;
		transform-origin: right center;
		transform: rotate(-45deg);
	}
	
	.star-points .star {
		position: absolute;
		width: 12rpx;
		height: 12rpx;
		background: #333;
		border-radius: 50%;
		margin-left: 20rpx;
		margin-top: 20rpx;
	}
	
	.board-overlay {
		position: absolute;
		top: 0;
		left: 0;
		width: 100%;
		height: 100%;
	}

	.board-cell {
		position: absolute;
		width: 60rpx;
		height: 60rpx;
		display: flex;
		align-items: center;
		justify-content: center;
		z-index: 2;
		margin-left: -4rpx;
		margin-top: -4rpx;
	}
	
	.board-cell.highlight {
		background: rgba(76, 175, 80, 0.3);
	}
	
	.board-cell.highlight::after {
		content: '';
		position: absolute;
		width: 20rpx;
		height: 20rpx;
		background: rgba(76, 175, 80, 0.6);
		border-radius: 50%;
	}
	
	.board-cell.selected {
		background: rgba(255, 215, 0, 0.3);
	}
	
	.piece {
		width: 52rpx;
		height: 52rpx;
		border-radius: 50%;
		display: flex;
		align-items: center;
		justify-content: center;
		position: relative;
		z-index: 3;
		transition: transform 0.15s, box-shadow 0.15s;
	}
	
	.piece.black {
		background: radial-gradient(circle, #555 0%, #222 50%, #111 100%);
		border: 3rpx solid #666;
		box-shadow: 2rpx 2rpx 4rpx rgba(0,0,0,0.5);
	}
	
	.piece.red {
		background: radial-gradient(circle, #FF6B6B 0%, #DC143C 50%, #8B0000 100%);
		border: 3rpx solid #FF8A8A;
		box-shadow: 2rpx 2rpx 4rpx rgba(0,0,0,0.5);
	}
	
	.piece.selected {
		transform: scale(1.1);
		box-shadow: 0 0 20rpx rgba(255, 215, 0, 0.8);
	}
	
	.piece-text {
		font-size: 26rpx;
		font-weight: bold;
		font-family: 'STKaiti', 'Kaiti SC', '楷体', serif;
	}
	
	.black .piece-text {
		color: #FFD700;
		text-shadow: 1rpx 1rpx 2rpx rgba(0,0,0,0.5);
	}
	
	.red .piece-text {
		color: white;
		text-shadow: 1rpx 1rpx 2rpx rgba(0,0,0,0.5);
	}
	
	.river-text {
		position: absolute;
		top: 230rpx;
		left: 0;
		right: 0;
		height: 60rpx;
		display: flex;
		justify-content: space-between;
		align-items: center;
		padding: 0 10rpx;
	}
	
	.river-text text {
		font-size: 24rpx;
		font-weight: bold;
		color: rgba(139, 69, 19, 0.6);
		font-family: 'STKaiti', 'Kaiti SC', '楷体', serif;
	}
	
	.action-bar {
		display: flex;
		justify-content: center;
		gap: 30rpx;
		padding: 30rpx;
	}
	
	.action-btn {
		padding: 20rpx 40rpx;
		background: rgba(255,255,255,0.95);
		border-radius: 30rpx;
		box-shadow: 0 4rpx 15rpx rgba(0,0,0,0.1);
	}
	
	.action-btn text {
		font-size: 28rpx;
		color: #2C5F2D;
		font-weight: bold;
	}
	
	.online-lobby {
		position: fixed;
		top: 50%;
		left: 50%;
		transform: translate(-50%, -50%);
		background: white;
		border-radius: 20rpx;
		padding: 40rpx;
		display: flex;
		flex-direction: column;
		align-items: center;
		box-shadow: 0 20rpx 50rpx rgba(0,0,0,0.3);
	}
	
	.lobby-title text {
		font-size: 32rpx;
		font-weight: bold;
		color: #333;
		margin-bottom: 30rpx;
	}
	
	.room-code {
		display: flex;
		flex-direction: column;
		align-items: center;
		margin-bottom: 30rpx;
	}
	
	.code-label {
		font-size: 24rpx;
		color: #999;
		margin-bottom: 10rpx;
	}
	
	.code-value {
		font-size: 48rpx;
		font-weight: bold;
		color: #2C5F2D;
		letter-spacing: 10rpx;
	}
	
	.copy-btn, .cancel-btn {
		padding: 20rpx 60rpx;
		border-radius: 30rpx;
		margin-bottom: 15rpx;
		width: 100%;
		text-align: center;
	}
	
	.copy-btn {
		background: #2C5F2D;
	}
	
	.copy-btn text {
		color: white;
		font-size: 28rpx;
	}
	
	.cancel-btn {
		background: #f0f0f0;
	}
	
	.cancel-btn text {
		color: #666;
		font-size: 28rpx;
	}
	
	.game-over-modal {
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
		display: flex;
		flex-direction: column;
		align-items: center;
	}
	
	.result-text {
		font-size: 40rpx;
		font-weight: bold;
		color: #2C5F2D;
		margin-bottom: 40rpx;
	}
	
	.modal-btns {
		display: flex;
		gap: 30rpx;
	}
	
	.modal-btn {
		padding: 20rpx 50rpx;
		background: #2C5F2D;
		border-radius: 30rpx;
	}
	
	.modal-btn.secondary {
		background: #f0f0f0;
	}
	
	.modal-btn text {
		font-size: 28rpx;
		color: white;
	}
	
	.modal-btn.secondary text {
		color: #666;
	}
</style>