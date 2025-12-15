<template>
	<view class="about-page">
		<view class="aboutPage pageBg"></view>
		
		<scroll-view class="content-scroll" scroll-y>
			<view class="content">
				<view class="card">
					<image class="logo-image" src="/static/logo.png" mode="aspectFit"></image>
					<view class="title">关于本项目</view>
					
					<view class="section">
						<text class="description">
							这是一个基于 uni-app 开发的 Minecraft 服务器配套小程序，支持多平台部署： Web、微信小程序、QQ小程序。
						</text>
					</view>
					
					<view class="section">
						<view class="section-title">✨ 主要功能</view>
						<view class="features-list">
							<view class="feature-item">
								<text class="feature-icon">🎨</text>
								<view class="feature-content">
									<text class="feature-name">服务器海报</text>
									<text class="feature-desc">精美的服务器宣传海报展示</text>
								</view>
							</view>
							<view class="feature-item">
								<text class="feature-icon">🏆</text>
								<view class="feature-content">
									<text class="feature-name">计分板查询</text>
									<text class="feature-desc">实时查看跑酷排行榜数据</text>
								</view>
							</view>
							<view class="feature-item">
								<text class="feature-icon">👥</text>
								<view class="feature-content">
									<text class="feature-name">玩家数据库</text>
									<text class="feature-desc">查看 Authme 登录记录</text>
								</view>
							</view>
							<view class="feature-item">
								<text class="feature-icon">📖</text>
								<view class="feature-content">
									<text class="feature-name">Markdown 指南</text>
									<text class="feature-desc">服务器游玩指南与教程</text>
								</view>
							</view>
							<view class="feature-item">
								<text class="feature-icon">🖥️</text>
								<view class="feature-content">
									<text class="feature-name">服务器状态</text>
									<text class="feature-desc">实时监控服务器在线状态</text>
								</view>
							</view>
						</view>
					</view>
					
					<view class="section">
						<view class="section-title">GitHub 仓库</view>
						<view class="github-link" @click="handleGitHubClick">
							<image 
								class="github-icon" 
								src="../../static/github-mark.png" 
								mode="aspectFit"
							></image>
							<text class="link-text">ichozero/uniapp-mp-qwq-server-frontend</text>
							<text class="action-hint">{{ actionHint }}</text>
						</view>
					</view>
					
					<view class="section">
						<view class="repo-url">
							<text class="url-text">{{ repoUrl }}</text>
						</view>
					</view>
				</view>
			</view>
		</scroll-view>

		<my-nav class="down-nav" :y="Number(5)" :x="XRef" />
	</view>
</template>

<script setup>
import { ref, computed } from 'vue'
import { onLoad } from '@dcloudio/uni-app'

const repoUrl = 'https://github.com/ichozero/uniapp-mp-qwq-server-frontend'

// Platform-specific action hint
const actionHint = computed(() => {
	// #ifdef H5
	return '点击访问'
	// #endif
	// #ifdef MP-WEIXIN || MP-QQ
	return '点击复制'
	// #endif
	// #ifndef H5 || MP-WEIXIN || MP-QQ
	return '点击复制'
	// #endif
})

function handleGitHubClick() {
	// #ifdef H5
	// Web platform: open in new tab
	window.open(repoUrl, '_blank')
	// #endif
	
	// #ifdef MP-WEIXIN || MP-QQ
	// Mini program: copy to clipboard
	uni.setClipboardData({
		data: repoUrl,
		success: () => {
			uni.showToast({
				title: '已复制到剪贴板',
				icon: 'success',
				duration: 2000
			})
		},
		fail: () => {
			uni.showToast({
				title: '复制失败',
				icon: 'none',
				duration: 2000
			})
		}
	})
	// #endif
	
	// #ifndef H5 || MP-WEIXIN || MP-QQ
	// Other platforms: copy to clipboard as fallback
	uni.setClipboardData({
		data: repoUrl,
		success: () => {
			uni.showToast({
				title: '已复制到剪贴板',
				icon: 'success',
				duration: 2000
			})
		},
		fail: () => {
			uni.showToast({
				title: '复制失败',
				icon: 'none',
				duration: 2000
			})
		}
	})
	// #endif
}

// navbar index handling
const XRef = ref(Number(5))
onLoad((event) => {
	XRef.value = event?.LastIdx ? Number(event.LastIdx) : 5
})
</script>

<style lang="scss" scoped>
.about-page { 
	height: 100vh; 
	width: 100%; 
}

.aboutPage {
	position: fixed;
	left: 0;
	top: 0;
	right: 0;
	bottom: 0;
	width: 100%;
	height: 100vh;
	z-index: -1;
}

.pageBg {
	background: linear-gradient(135deg, #1e3c72 0%, #2a5298 25%, #7e22ce 50%, #ec4899 75%, #f472b6 100%);
	background-size: auto;
	background-position: center;
}

.content-scroll { 
	flex: 1; 
	height: 100%; 
}

.content { 
	padding: 40rpx 30rpx;
	padding-bottom: 140rpx;
	display: flex;
	justify-content: center;
	align-items: flex-start;
	min-height: calc(100vh - 140rpx);
}

.card {
	background: rgba(255, 255, 255, 0.95);
	backdrop-filter: blur(10px);
	border-radius: 24rpx;
	padding: 40rpx;
	box-shadow: 0 8rpx 32rpx rgba(0, 0, 0, 0.15);
	width: 100%;
	max-width: 680rpx;
}

.logo-image {
	width: 160rpx;
	height: 160rpx;
	display: block;
	margin: 0 auto 24rpx auto;
	border-radius: 20rpx;
	box-shadow: 0 4rpx 16rpx rgba(0, 0, 0, 0.1);
	transition: all 0.4s cubic-bezier(0.175, 0.885, 0.32, 1.275);
	cursor: pointer;
}

.logo-image:hover {
	transform: scale(1.15) rotateY(8deg) rotateX(-8deg);
	box-shadow: 0 20rpx 60rpx rgba(102, 126, 234, 0.4),
	            0 0 40rpx rgba(236, 72, 153, 0.3),
	            inset 0 0 20rpx rgba(255, 255, 255, 0.2);
}

.title {
	font-size: 48rpx;
	font-weight: 700;
	color: #333;
	margin-bottom: 32rpx;
	text-align: center;
}

.section {
	margin-bottom: 32rpx;
}

.section-title {
	font-size: 32rpx;
	font-weight: 600;
	color: #555;
	margin-bottom: 16rpx;
}

.description {
	font-size: 28rpx;
	color: #666;
	line-height: 1.6;
}

.features-list {
	display: flex;
	flex-direction: column;
	gap: 16rpx;
}

.feature-item {
	display: flex;
	align-items: flex-start;
	padding: 20rpx;
	background: #f8f9fa;
	border-radius: 12rpx;
	transition: all 0.3s cubic-bezier(0.175, 0.885, 0.32, 1.275);
	cursor: pointer;
}

.feature-item:hover {
	transform: scale(1.05) translateY(-4rpx) rotateX(2deg);
	background: linear-gradient(135deg, #f8f9fa 0%, #e8eaf6 100%);
	box-shadow: 0 12rpx 32rpx rgba(102, 126, 234, 0.2),
	            0 0 30rpx rgba(118, 75, 162, 0.15),
	            inset 0 2rpx 4rpx rgba(255, 255, 255, 0.8);
}

.feature-item:active {
	transform: scale(0.98) translateX(4rpx);
	box-shadow: 0 2rpx 8rpx rgba(0, 0, 0, 0.08);
}

.feature-icon {
	font-size: 40rpx;
	margin-right: 16rpx;
	flex-shrink: 0;
	line-height: 1;
}

.feature-content {
	flex: 1;
	display: flex;
	flex-direction: column;
	gap: 4rpx;
}

.feature-name {
	font-size: 28rpx;
	font-weight: 600;
	color: #333;
.github-link {
	display: flex;
	align-items: center;
	padding: 24rpx;
	background: linear-gradient(135deg, #f5f7fa 0%, #c3cfe2 100%);
	border-radius: 16rpx;
	cursor: pointer;
	transition: all 0.35s cubic-bezier(0.175, 0.885, 0.32, 1.275);
	box-shadow: 0 4rpx 12rpx rgba(0, 0, 0, 0.08);
	gap: 16rpx;
	flex-wrap: wrap;
}

.github-link:hover {
	transform: scale(1.08) translateY(-6rpx) rotateY(-3deg) rotateX(3deg);
	background: linear-gradient(135deg, #e3f2fd 0%, #bbdefb 100%);
	box-shadow: 0 20rpx 50rpx rgba(33, 150, 243, 0.3),
	            0 0 40rpx rgba(102, 126, 234, 0.25),
	            inset 0 2rpx 8rpx rgba(255, 255, 255, 0.6);
}

.github-link:active {
	transform: scale(0.98);
	box-shadow: 0 2rpx 8rpx rgba(0, 0, 0, 0.12);
}transition: transform 0.2s, box-shadow 0.2s;
	box-shadow: 0 4rpx 12rpx rgba(0, 0, 0, 0.08);
	gap: 16rpx;
	flex-wrap: wrap;
}

.github-link:active {
	transform: scale(0.98);
	box-shadow: 0 2rpx 8rpx rgba(0, 0, 0, 0.12);
}

.github-icon {
	width: 48rpx;
	height: 48rpx;
	flex-shrink: 0;
}

.link-text {
	font-size: 28rpx;
	color: #333;
	font-weight: 500;
	flex: 1;
	word-break: break-word;
}

.action-hint {
	font-size: 24rpx;
	color: #667eea;
	padding: 4rpx 12rpx;
	background: rgba(102, 126, 234, 0.1);
	border-radius: 8rpx;
	white-space: nowrap;
}

.repo-url {
	padding: 20rpx;
	background: #f8f9fa;
	border-radius: 12rpx;
	border-left: 6rpx solid #667eea;
}

.url-text {
	font-size: 24rpx;
	color: #666;
	word-break: break-all;
	font-family: 'Courier New', monospace;
}

@media (max-width: 480px) {
	.card {
		padding: 32rpx 24rpx;
	}
	
	.title {
		font-size: 40rpx;
	}
	
	.github-link {
		padding: 20rpx;
	}
}

.down-nav { 
	position: fixed; 
	bottom: 10px; 
	left: 0; 
}
</style>
