<template>
  <view class="container">
    <view class="text-wrapper">
      <text class="title-cn">施工中</text>
      <text class="title-en">Under Construction</text>
    </view>
    <image class="image" src="../../static/mc_lucky_block.jpg" mode="aspectFill" />
  </view>
  
  <my-nav class="down-nav" :y="Number(3)" :x="XRef"></my-nav>
</template>

<script setup>
	import { ref, onMounted } from 'vue';
	import { onLoad } from "@dcloudio/uni-app";
	import { getCurrentIdx } from "@/utils/page.js"
// 暂无逻辑，这里可以添加你想要的逻辑
	// ---------------------------------------------
	const XRef = ref( Number(3) );
	
	onLoad((event) => {
		XRef.value = event.LastIdx ? Number(event.LastIdx) : 3;
	});
</script>

<style lang="scss" scoped>
.container {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  height: 100vh;
  background: linear-gradient(to bottom right, #f8f9fa, #e9ecef);
  padding: 30rpx;
}

.text-wrapper {
  text-align: center;
  margin-bottom: 40rpx;
}

.title-cn {
  font-size: 60rpx;
  font-weight: bold;
  color: #ff4d4f;
}

.title-en {
  font-size: 36rpx;
  color: #666;
  margin-top: 10rpx;
}

.image {
  width: 500rpx;
  height: 500rpx;
  border-radius: 20rpx;
  box-shadow: 0 10rpx 20rpx rgba(0, 0, 0, 0.1);
}

.down-nav{
		position: fixed;
		bottom: 10px;
		left: 0;
	}
</style>
