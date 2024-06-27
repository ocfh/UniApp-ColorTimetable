<script setup lang="ts">
	
const { darkMode, statusBarHeight, menuButtonBounding } = storeToRefs(useAppStore())
import { onMounted, onUnmounted } from 'vue';  
  
// 页面加载错误处理函数  
const handlePageError = () => {  
  uni.redirectTo({ url: '/pages/index/index' });  
};  
  
// 在组件挂载后设置全局错误处理函数  
onMounted(() => {  
  const originalOnError = uni.onError;  
  uni.onError = (error) => {  
    if (error.errMsg.indexOf('fail') !== -1) {  
      handlePageError();  
    } else {  
      // 调用原来的 onError 处理函数  
      originalOnError(error);  
    }  
  };  
});  
  
// 在组件卸载前恢复全局错误处理函数  
onUnmounted(() => {  
  uni.onError = originalOnError;  
});  
onLaunch(() => {
	let style = document.createElement('style');
	style.innerHTML = `  
	.uni-loading,
	uni-button[loading]:before,uni-loading,uni-async-loading {
	  background-size: cover!important;
	  min-height:50px;
	  min-width:46px;
	  animation: shake 0.5s infinite!important;
	}@keyframes shake {  
  0% { transform: translate(1px, 0); }  
  10% { transform: translate(-1px, -2px); }  
  20% { transform: translate(-3px, 0); }  
  30% { transform: translate(1px, 2px); }  
  40% { transform: translate(1px, 0); }  
  50% { transform: translate(-1px, -2px); }  
  60% { transform: translate(-3px, 0); }  
  70% { transform: translate(1px, 2px); }  
  80% { transform: translate(1px, 0); }  
  90% { transform: translate(-1px, -2px); }  
  100% { transform: translate(0, 0); }  
}  `;  
	  
	document.head.appendChild(style);
  // #ifdef MP-WEIXIN || MP-QQ
  const systemInfo = uni.getSystemInfoSync()
  // the systemInfo.theme is only support dark mode in WeChat and QQ
  darkMode.value = systemInfo?.theme === 'dark'
  statusBarHeight.value = systemInfo!.statusBarHeight || 44
  menuButtonBounding.value = uni.getMenuButtonBoundingClientRect()
  uni.onThemeChange((res: UniApp.OnThemeChangeCallbackResult) => darkMode.value = res.theme === 'dark')
  // #endif

  // #ifdef H5
  const colorScheme = window.matchMedia('(prefers-color-scheme: dark)')
  darkMode.value = colorScheme.matches
  colorScheme.addEventListener('change', (e: MediaQueryListEvent) => darkMode.value = e.matches)
  // The data is obtained from iPhone13 miniprogram but statusBarHeight, top and bottom values are subtracted from the statusBarHeight value
  statusBarHeight.value = 0
  menuButtonBounding.value = { width: 87, height: 32, left: 281, top: 4, right: 368, bottom: 36 }
  // #endif
})
onShow(() => {
})
onHide(() => {
})

</script>

