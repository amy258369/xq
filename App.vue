<script>
	export default {
		onLaunch: function() {
			console.log('App Launch')
		},
		onShow: function() {
			console.log('App Show')
		},
		onHide: function() {
			console.log('App Hide')
		}
	}
</script>

<style>
	page {
		background-color: #F5F5DC;
		font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, 'Helvetica Neue', Arial, sans-serif;
	}
	.container {
		min-height: 100vh;
		padding: 0;
		margin: 0;
	}
	@keyframes fadeIn {
		from { opacity: 0; transform: translateY(20px); }
		to { opacity: 1; transform: translateY(0); }
	}
	@keyframes bounce {
		0%, 20%, 50%, 80%, 100% { transform: translateY(0); }
		40% { transform: translateY(-20px); }
		60% { transform: translateY(-10px); }
	}
	@keyframes shake {
		0%, 100% { transform: translateX(0); }
		10%, 30%, 50%, 70%, 90% { transform: translateX(-5px); }
		20%, 40%, 60%, 80% { transform: translateX(5px); }
	}
	.animate-fadeIn {
		animation: fadeIn 0.5s ease-out;
	}
	.animate-bounce {
		animation: bounce 1s ease-in-out;
	}
	.animate-shake {
		animation: shake 0.5s ease-in-out;
	}
</style>