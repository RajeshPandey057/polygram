<script lang="ts">
	import { onMount } from "svelte";
	import { goto } from "$app/navigation";
	import SEO from "@/components/seo.svelte";
	import LoginButton from "$lib/components/login-button.svelte";
	import { authStore } from "$lib/stores/auth-store.svelte";

	const gradientBackground =
		"radial-gradient(circle at 25% 10%, rgba(255,255,255,0.08), rgba(8,8,8,0.9)), linear-gradient(135deg, #040404 0%, #0E0E0E 45%, #020202 100%)";

	onMount(() => {
		// If already authenticated, redirect to home
		if (authStore.isAuthenticated) {
			goto("/");
		}
	});

	function handleLogin() {
		// Mock login - just set a user
		authStore.login();
		// Redirect to home after login
		goto("/");
	}
</script>

<SEO
	title="Login - Polygram"
	description="Sign in to Polygram to access financial news, insights, and analytics"
/>

<main class="flex min-h-screen items-center justify-center text-white" style={`background:${gradientBackground};`}>
	<div class="mx-auto flex w-full max-w-md flex-col items-center gap-8 px-6 py-12">
		<!-- Polygram Logo/Header -->
		<div class="flex flex-col items-center gap-4">
			<div class="flex size-16 items-center justify-center rounded-full bg-white/20">
				<!-- Placeholder icon - light gray circle -->
			</div>
			<h1 class="text-3xl font-semibold tracking-tight text-white">Polygram</h1>
			<p class="text-center text-sm text-white/70">
				AI-powered financial news and insights
			</p>
		</div>

		<!-- Login Card -->
		<div class="w-full rounded-2xl border border-white/10 bg-black/70 p-8 shadow-[0px_20px_50px_rgba(0,0,0,0.45)]">
			<div class="flex flex-col gap-6">
				<div class="text-center">
					<h2 class="text-xl font-semibold text-white">Welcome back</h2>
					<p class="mt-2 text-sm text-white/60">
						Sign in to continue to Polygram
					</p>
				</div>

				<LoginButton onLogin={handleLogin} />
			</div>
		</div>

		<!-- Footer text -->
		<p class="text-xs text-white/50">
			By continuing, you agree to Polygram's Terms of Service and Privacy Policy
		</p>
	</div>
</main>
