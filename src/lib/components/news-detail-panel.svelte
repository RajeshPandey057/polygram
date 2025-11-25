<script lang="ts">
	import { Button } from "$lib/components/ui/button/index.js";
	import X from "~icons/lucide/x";
	import Send from "~icons/lucide/send-horizontal";
	import TrendingUp from "~icons/lucide/trending-up";
	import Minus from "~icons/lucide/minus";
	import TrendingDown from "~icons/lucide/trending-down";
	import ThumbsUp from "~icons/lucide/thumbs-up";
	import LinkIcon from "~icons/lucide/link";
	import Bot from "~icons/lucide/bot";
	import User from "~icons/lucide/user";
	import { goto } from "$app/navigation";
	import { page } from "$app/stores";
	import type { NewsDetail } from "$lib/types/news";

	let {
		open = $bindable(false),
		detail = $bindable<NewsDetail | null>(null),
	}: {
		open?: boolean;
		detail: NewsDetail | null;
	} = $props();

	let selectedTimeframe = $state<"1D" | "1W" | "1M" | "1Y">("1D");
	let aiInput = $state("");
	let chatMode = $state(false);
	let messages = $state<Array<{ role: "user" | "assistant"; content: string; timestamp: Date }>>([]);
	let loading = $state(false);
	let contentRef: HTMLDivElement | null = $state(null);

	const changeValue = $derived(
		detail ? detail.changes[selectedTimeframe] : 0
	);

	const changeTone = $derived(changeValue >= 0 ? "positive" : "negative");

	function getImpactColor(level: string) {
		if (level.includes("Very Positive")) return "bg-green-500";
		if (level.includes("Positive")) return "bg-green-400";
		if (level.includes("Very Negative")) return "bg-red-500";
		if (level.includes("Negative")) return "bg-red-400";
		return "bg-yellow-400";
	}

	async function handleSendMessage() {
		const messageToSend = aiInput.trim();
		if (!messageToSend) return;

		// Switch to chat mode on first message
		if (!chatMode) {
			chatMode = true;
		}

		// Add user message
		messages = [
			...messages,
			{
				role: "user",
				content: messageToSend,
				timestamp: new Date(),
			},
		];
		aiInput = "";

		// Simulate AI response
		loading = true;
		await new Promise((resolve) => setTimeout(resolve, 1000));

		messages = [
			...messages,
			{
				role: "assistant",
				content: `I understand you're asking about: "${messageToSend}" regarding ${detail?.name || "this news"}. This is a placeholder response. The AI chat functionality will be integrated with your backend service.`,
				timestamp: new Date(),
			},
		];

		loading = false;

		// Scroll to bottom to show latest message
		setTimeout(() => {
			if (contentRef) {
				contentRef.scrollTop = contentRef.scrollHeight;
			}
		}, 100);
	}

	function handleKeydown(event: KeyboardEvent) {
		if (event.key === "Enter" && !event.shiftKey) {
			event.preventDefault();
			handleSendMessage();
		}
	}

	function formatTime(date: Date): string {
		return date.toLocaleTimeString("en-US", {
			hour: "numeric",
			minute: "2-digit",
			hour12: true,
		});
	}

	// Reset chat mode when panel closes
	$effect(() => {
		if (!open) {
			chatMode = false;
			messages = [];
			aiInput = "";
		}
	});
</script>

{#if open && detail}
	<!-- Overlay -->
	<button
		type="button"
		class="fixed inset-0 z-40 bg-black/50"
		onclick={() => (open = false)}
		onkeydown={(e) => e.key === "Escape" && (open = false)}
		aria-label="Close panel"
	></button>

		<!-- Panel -->
		<div
			class="fixed right-0 top-0 z-50 h-full w-full max-w-2xl overflow-y-auto border-l border-white/7 bg-[#171717] text-white shadow-2xl"
			role="dialog"
			aria-modal="true"
			aria-labelledby="news-detail-title"
		>
		<!-- Header -->
		<div class="sticky top-0 z-10 flex items-center justify-between border-b border-white/7 bg-[#171717] px-4 py-4">
			<button
				type="button"
				id="news-detail-title"
				onclick={() => {
					// Include return URL so back button works correctly
					const currentPath = $page.url.pathname;
					goto(`/stocks/${detail.code}?return=${encodeURIComponent(currentPath)}`);
					open = false;
				}}
				class="text-base font-medium leading-normal text-white transition hover:text-[#8CFF88] cursor-pointer"
			>
				{detail.name}
			</button>
			<button
				type="button"
				onclick={() => (open = false)}
				class="flex size-8 items-center justify-center rounded p-2 text-white/70 transition-opacity hover:opacity-100 hover:text-white"
				aria-label="Close panel"
			>
				<X class="size-4" />
			</button>
		</div>

		<!-- Content -->
		<div
			bind:this={contentRef}
			class="flex max-h-[calc(100vh-120px)] flex-col gap-9 overflow-y-auto px-4 py-0 transition-all duration-300"
		>
			<!-- News Summary -->
			<div class="flex flex-col gap-4">
				<div class="flex h-[30px] items-center px-2">
					<p class="text-xs font-medium leading-[12px] text-[#91918e]">News Summary</p>
				</div>
				<p class="text-sm leading-relaxed text-white/80">{detail.summary}</p>
			</div>

			<!-- Possible Impact -->
			<div class="flex flex-col gap-4">
				<div class="flex h-[30px] items-center px-2">
					<p class="text-xs font-medium leading-[12px] text-[#91918e]">Possible Impact</p>
				</div>
				<div class="flex flex-col gap-4">
					<div class="h-2 w-full overflow-hidden rounded-full bg-white/10">
						<div
							class="h-full transition-all {getImpactColor(detail.marketImpact.level)}"
							style={`width: ${detail.marketImpact.percentage}%`}
						></div>
					</div>
					<div class="flex items-center gap-2">
						<button
							type="button"
							onclick={() => (selectedTimeframe = "1D")}
							class={`rounded border px-3 py-1.5 text-sm font-medium transition ${
								selectedTimeframe === "1D"
									? "border-white/7 bg-[rgba(0,143,29,0.18)] text-[rgba(140,255,136,0.7)]"
									: "border-white/7 bg-white/4 text-white/70 hover:bg-white/8"
							}`}
						>
							1D
						</button>
						<button
							type="button"
							onclick={() => (selectedTimeframe = "1W")}
							class={`rounded border px-3 py-1.5 text-sm font-medium transition ${
								selectedTimeframe === "1W"
									? "border-white/7 bg-[rgba(0,143,29,0.18)] text-[rgba(140,255,136,0.7)]"
									: "border-white/7 bg-white/4 text-white/70 hover:bg-white/8"
							}`}
						>
							1W
						</button>
						<button
							type="button"
							onclick={() => (selectedTimeframe = "1M")}
							class={`rounded border px-3 py-1.5 text-sm font-medium transition ${
								selectedTimeframe === "1M"
									? "border-white/7 bg-[rgba(0,143,29,0.18)] text-[rgba(140,255,136,0.7)]"
									: "border-white/7 bg-white/4 text-white/70 hover:bg-white/8"
							}`}
						>
							1M
						</button>
						<button
							type="button"
							onclick={() => (selectedTimeframe = "1Y")}
							class={`rounded border px-3 py-1.5 text-sm font-medium transition ${
								selectedTimeframe === "1Y"
									? "border-white/7 bg-[rgba(0,143,29,0.18)] text-[rgba(140,255,136,0.7)]"
									: "border-white/7 bg-white/4 text-white/70 hover:bg-white/8"
							}`}
						>
							1Y
						</button>
						<span
							class={`ml-2 text-sm font-semibold ${
								changeTone === "positive" ? "text-[#8CFF88]" : "text-[#FFA8A8]"
							}`}
						>
							{changeTone === "positive" ? "+" : ""}
							{changeValue.toFixed(1)}% in last {selectedTimeframe === "1D" ? "1 day" : selectedTimeframe === "1W" ? "1 week" : selectedTimeframe === "1M" ? "1 month" : "1 year"}
						</span>
					</div>
				</div>
			</div>

			<!-- Investor's Mood Today -->
			<div class="flex flex-col gap-4">
				<div class="flex h-[30px] items-center px-2">
					<p class="text-xs font-medium leading-[12px] text-[#91918e]">Investor's Mood Today</p>
				</div>
				<div class="flex flex-col gap-2">
					<div class="flex flex-col gap-1">
						<div class="flex items-center justify-between text-sm">
							<div class="flex items-center gap-2">
								<TrendingUp class="size-4 text-green-400" />
								<span class="text-white/80">Bullish</span>
							</div>
							<span class="font-medium text-white">{detail.investorMood.bullish}%</span>
						</div>
						<div class="h-2 w-full overflow-hidden rounded-full bg-white/10">
							<div
								class="h-full bg-green-500"
								style={`width: ${detail.investorMood.bullish}%`}
							></div>
						</div>
					</div>
					<div class="flex flex-col gap-1">
						<div class="flex items-center justify-between text-sm">
							<div class="flex items-center gap-2">
								<Minus class="size-4 text-yellow-400" />
								<span class="text-white/80">Neutral</span>
							</div>
							<span class="font-medium text-white">{detail.investorMood.neutral}%</span>
						</div>
						<div class="h-2 w-full overflow-hidden rounded-full bg-white/10">
							<div
								class="h-full bg-yellow-400"
								style={`width: ${detail.investorMood.neutral}%`}
							></div>
						</div>
					</div>
					<div class="flex flex-col gap-1">
						<div class="flex items-center justify-between text-sm">
							<div class="flex items-center gap-2">
								<TrendingDown class="size-4 text-red-400" />
								<span class="text-white/80">Bearish</span>
							</div>
							<span class="font-medium text-white">{detail.investorMood.bearish}%</span>
						</div>
						<div class="h-2 w-full overflow-hidden rounded-full bg-white/10">
							<div
								class="h-full bg-red-500"
								style={`width: ${detail.investorMood.bearish}%`}
							></div>
						</div>
					</div>
				</div>
			</div>

			<!-- Dominant Phrase -->
			<div class="flex flex-col gap-4">
				<div class="flex h-[30px] items-center px-2">
					<p class="text-xs font-medium leading-[12px] text-[#91918e]">
						Dominant Phrase across the news
					</p>
				</div>
				<div class="flex items-center gap-3 rounded border border-white/7 bg-white/4 p-4">
					<span class="text-3xl text-white/50">"</span>
					<p class="flex-1 text-lg font-medium text-[#8CFF88]">
						{detail.dominantPhrase}
					</p>
					<span class="text-3xl text-white/50">"</span>
				</div>
			</div>

			<!-- All Headlines -->
			<div class="flex flex-col gap-4">
				<div class="flex h-[30px] items-center px-2">
					<p class="text-xs font-medium leading-[12px] text-[#91918e]">All Headlines</p>
				</div>
				<div class="flex flex-col gap-4">
					{#each detail.citations as citation}
						<div class="flex flex-col gap-2 rounded border border-white/7 bg-white/4 p-4">
							<h4 class="font-semibold text-white">{citation.header}</h4>
							<p class="text-sm leading-relaxed text-white/70">{citation.summary}</p>
							<div class="flex flex-wrap items-center gap-3 text-xs text-white/50">
								<span>{citation.date}, {citation.time}</span>
								<div class="flex items-center gap-1">
									<ThumbsUp class="size-3" />
									<span
										class={
											citation.sentiment === "Positive"
												? "text-green-400"
												: citation.sentiment === "Negative"
													? "text-red-400"
													: "text-yellow-400"
										}
									>
										{citation.sentiment}
									</span>
								</div>
								<div class="flex items-center gap-1">
									<LinkIcon class="size-3" />
									<span>{citation.channel}</span>
								</div>
							</div>
						</div>
					{/each}
				</div>
			</div>

			<!-- Chat Messages (appears below news content when chat mode is active) -->
			{#if chatMode}
				<div class="flex flex-col gap-4 border-t border-white/7 pt-6">
					<div class="flex h-[30px] items-center px-2">
						<p class="text-xs font-medium leading-[12px] text-[#91918e]">AI Conversation</p>
					</div>
					<div class="flex flex-col gap-4">
						{#if messages.length === 0}
							<div class="flex items-center justify-center py-8">
								<div class="text-center">
									<Bot class="mx-auto mb-4 size-12 text-white/40" />
									<p class="text-sm text-white/60">Start a conversation about this news</p>
								</div>
							</div>
						{:else}
							{#each messages as message (message.timestamp.getTime())}
								<div
									class="flex gap-3 {message.role === 'user' ? 'flex-row-reverse' : 'flex-row'}"
								>
									<div
										class="flex size-8 shrink-0 items-center justify-center rounded-full {message.role === 'user'
											? 'bg-white/20'
											: 'bg-white/10'}"
									>
										{#if message.role === "user"}
											<User class="size-4 text-white" />
										{:else}
											<Bot class="size-4 text-white" />
										{/if}
									</div>
									<div
										class="flex max-w-[80%] flex-col gap-1 {message.role === 'user' ? 'items-end' : 'items-start'}"
									>
										<div
											class="rounded-lg px-4 py-2.5 text-sm {message.role === 'user'
												? 'bg-white/10 text-white'
												: 'bg-white/5 text-white/90'}"
										>
											{message.content}
										</div>
										<span class="text-xs text-white/40">{formatTime(message.timestamp)}</span>
									</div>
								</div>
							{/each}
							{#if loading}
								<div class="flex gap-3">
									<div class="flex size-8 shrink-0 items-center justify-center rounded-full bg-white/10">
										<Bot class="size-4 text-white" />
									</div>
									<div class="flex flex-col gap-1">
										<div class="rounded-lg bg-white/5 px-4 py-2.5 text-sm text-white/90">
											<div class="flex gap-1">
												<span class="h-2 w-2 animate-bounce rounded-full bg-white/60 [animation-delay:-0.3s]"></span>
												<span class="h-2 w-2 animate-bounce rounded-full bg-white/60 [animation-delay:-0.15s]"></span>
												<span class="h-2 w-2 animate-bounce rounded-full bg-white/60"></span>
											</div>
										</div>
									</div>
								</div>
							{/if}
						{/if}
					</div>
				</div>
			{/if}
		</div>

		<!-- Deep dive with AI -->
		<div class="sticky bottom-0 border-t border-white/7 bg-[#171717] px-4 py-4 transition-all duration-300 {chatMode
			? 'pb-4'
			: ''}">
			<div class="flex items-center gap-2 rounded-xl border border-white/15 bg-[#2e2e2e] px-4 py-3">
				<input
					bind:value={aiInput}
					onkeydown={handleKeydown}
					type="text"
					placeholder={chatMode ? "Ask a follow-up question..." : "Deep dive with AI"}
					class="flex-1 border-0 bg-transparent text-white placeholder:text-white/50 focus:outline-none"
					disabled={loading}
				/>
				<button
					type="button"
					onclick={handleSendMessage}
					disabled={loading || !aiInput.trim()}
					class="rounded-full border border-white/20 bg-white/10 p-2 text-white transition hover:bg-white/20 disabled:opacity-50 disabled:cursor-not-allowed"
					aria-label="Send AI prompt"
				>
					<Send class="size-4" />
				</button>
			</div>
		</div>
	</div>
{/if}
