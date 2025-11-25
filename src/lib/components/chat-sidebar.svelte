<script lang="ts">
	import X from "~icons/lucide/x";
	import Send from "~icons/lucide/send-horizontal";
	import Bot from "~icons/lucide/bot";
	import User from "~icons/lucide/user";

	let {
		open = $bindable(false),
		initialMessage = "",
	}: {
		open?: boolean;
		initialMessage?: string;
	} = $props();

	let messages = $state<Array<{ role: "user" | "assistant"; content: string; timestamp: Date }>>([]);
	let inputValue = $state("");
	let loading = $state(false);
	let lastInitialMessage = $state("");

	// Add initial message if provided
	$effect(() => {
		if (open && initialMessage && initialMessage !== lastInitialMessage) {
			lastInitialMessage = initialMessage;
			messages = [
				{
					role: "user",
					content: initialMessage,
					timestamp: new Date(),
				},
			];
			// Simulate AI response
			handleSendMessage(initialMessage);
		}
	});

	// Reset when closed
	$effect(() => {
		if (!open) {
			messages = [];
			inputValue = "";
			lastInitialMessage = "";
		}
	});

	async function handleSendMessage(message?: string) {
		const messageToSend = message || inputValue.trim();
		if (!messageToSend) return;

		// Add user message
		if (!message) {
			messages = [
				...messages,
				{
					role: "user",
					content: messageToSend,
					timestamp: new Date(),
				},
			];
			inputValue = "";
		}

		// Simulate AI response
		loading = true;
		await new Promise((resolve) => setTimeout(resolve, 1000));

		messages = [
			...messages,
			{
				role: "assistant",
				content: `I understand you're asking about: "${messageToSend}". This is a placeholder response. The AI chat functionality will be integrated with your backend service.`,
				timestamp: new Date(),
			},
		];

		loading = false;
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
</script>

{#if open}
	<!-- Overlay -->
	<button
		type="button"
		class="fixed inset-0 z-40 bg-black/50"
		onclick={() => (open = false)}
		onkeydown={(e) => e.key === "Escape" && (open = false)}
		aria-label="Close chat"
	></button>

	<!-- Chat Sidebar -->
	<div
		class="fixed right-0 top-0 z-50 h-full w-full max-w-2xl overflow-hidden border-l border-white/7 bg-[#171717] text-white shadow-2xl"
		role="dialog"
		aria-modal="true"
		aria-labelledby="chat-title"
	>
		<!-- Header -->
		<div class="sticky top-0 z-10 flex items-center justify-between border-b border-white/7 bg-[#171717] px-4 py-4">
			<div class="flex items-center gap-3">
				<div class="flex size-10 items-center justify-center rounded-full bg-white/20">
					<Bot class="size-5 text-white" />
				</div>
				<div>
					<h2 id="chat-title" class="text-base font-semibold text-white">AI Assistant</h2>
					<p class="text-xs text-white/60">Ask questions about today's news</p>
				</div>
			</div>
			<button
				type="button"
				onclick={() => (open = false)}
				class="flex size-8 items-center justify-center rounded p-2 text-white/70 transition-opacity hover:opacity-100 hover:text-white"
				aria-label="Close chat"
			>
				<X class="size-4" />
			</button>
		</div>

		<!-- Messages -->
		<div class="flex h-[calc(100vh-140px)] flex-col gap-4 overflow-y-auto px-4 py-6">
			{#if messages.length === 0}
				<div class="flex flex-1 items-center justify-center">
					<div class="text-center">
						<Bot class="mx-auto mb-4 size-12 text-white/40" />
						<p class="text-sm text-white/60">Start a conversation by asking about today's news</p>
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

		<!-- Input Area -->
		<div class="sticky bottom-0 border-t border-white/7 bg-[#171717] px-4 py-4">
			<div class="flex items-center gap-2 rounded-xl border border-white/15 bg-[#2e2e2e] px-4 py-3">
				<input
					bind:value={inputValue}
					onkeydown={handleKeydown}
					type="text"
					placeholder="Ask a follow-up question..."
					class="flex-1 border-0 bg-transparent text-white placeholder:text-white/50 focus:outline-none"
					disabled={loading}
				/>
				<button
					type="button"
					onclick={() => handleSendMessage()}
					disabled={loading || !inputValue.trim()}
					class="rounded-full border border-white/20 bg-white/10 p-2 text-white transition hover:bg-white/20 disabled:opacity-50 disabled:cursor-not-allowed"
					aria-label="Send message"
				>
					<Send class="size-4" />
				</button>
			</div>
		</div>
	</div>
{/if}

