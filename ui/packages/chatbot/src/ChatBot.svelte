<script lang="ts">
	import { beforeUpdate, afterUpdate, createEventDispatcher } from "svelte";
	import type { Styles } from "@gradio/utils";

	export let value: Array<[string | null, string | null]> | null;
	let old_value: Array<[string | null, string | null]> | null;
	export let pending_message: boolean = false;
	export let root: string;

	let div: HTMLDivElement;
	let autoscroll: Boolean;

	const dispatch = createEventDispatcher<{ change: undefined }>();
	const redirect_src_url = (src: string) =>
		src.replace('src="/file', `src="${root}file`);

	$: _value = value
		? value.map(([user_msg, bot_msg]) => [
				user_msg ? redirect_src_url(user_msg) : null,
				bot_msg ? redirect_src_url(bot_msg) : null
		  ])
		: [];
	beforeUpdate(() => {
		autoscroll =
			div && div.offsetHeight + div.scrollTop > div.scrollHeight - 20;
	});

	afterUpdate(() => {
		if (autoscroll) {
			div.scrollTo(0, div.scrollHeight);
			div.querySelectorAll("img").forEach((n) => {
				n.addEventListener("load", () => {
					div.scrollTo(0, div.scrollHeight);
				});
			});
		}
	});

	$: {
		if (value !== old_value) {
			old_value = value;
			dispatch("change");
		}
	}
</script>

<div class="wrap" bind:this={div}>
	<div class="message-wrap">
		{#each _value as message, i}
			<div
				data-testid="user"
				class:latest={i === _value.length - 1}
				class="message user"
				class:hide={message[0] === null}
			>
				{@html message[0]}
			</div>
			<div
				data-testid="bot"
				class:latest={i === _value.length - 1}
				class="message bot"
				class:hide={message[1] === null}
			>
				{@html message[1]}
			</div>
		{/each}
		{#if pending_message}
			<div data-testid="bot" class="message pending">
				<div class="dot-flashing" />
				&nbsp;
				<div class="dot-flashing" />
				&nbsp;
				<div class="dot-flashing" />
			</div>
		{/if}
	</div>
</div>

<style>
	.wrap {
		margin-top: var(--size-4);
		height: 100%;
		max-height: 480px;
		overflow-y: auto;
	}

	.message-wrap {
		display: flex;
		flex-direction: column;
		gap: var(--size-4);
		padding: var(--size-3);
	}

	.message-wrap > div :global(img) {
		border-radius: 13px;
		max-width: 30vw;
	}

	.message {
		border-width: 1px;
		border-style: solid;
		border-radius: var(--size-2);
		padding: var(--size-3);
		font-size: var(--scale-0);
		line-height: var(--line-md);
		overflow-wrap: break-word;
	}

	.user {
		margin-left: var(--size-6);
		border-color: var(--color-accent-light);
		border-bottom-right-radius: 0;
		background: var(--color-accent-soft);
		color: var(--color-text-body);
	}
	.pending,
	.bot {
		border-color: var(--color-border-primary);
		background: var(--color-background-secondary);
	}
	.bot {
		margin-right: var(--size-6);
		border-bottom-left-radius: 0;
		padding-left: var(--size-9);
	}
	.pending {
		margin: 0 var(--size-6);
	}
	:global(.dark) .user {
		border-color: var(--color-border-primary);
		background: var(--color-grey-700);
		color: var(--color-text-body);
	}

	.pending {
		display: flex;
		justify-content: center;
		align-items: center;
		gap: 2px;
	}
	.dot-flashing {
		animation: dot-flashing 1s infinite linear alternate;
		border-radius: 5px;
		background-color: var(--color-text-subdued);
		width: 5px;
		height: 5px;
		color: var(--color-text-subdued);
	}
	.dot-flashing:nth-child(2) {
		animation-delay: 0.33s;
	}
	.dot-flashing:nth-child(3) {
		animation-delay: 0.66s;
	}

	/* Small screen */
	@media (max-width: 480px) {
		.user {
			align-self: flex-end;
		}
		.bot {
			align-self: flex-start;
			padding-left: var(--size-3);
		}
	}

	@keyframes dot-flashing {
		0% {
			opacity: 0.8;
		}
		50% {
			opacity: 0.5;
		}
		100% {
			opacity: 0.8;
		}
	}
	.message-wrap .message :global(img) {
		margin: var(--size-2);
		max-height: 200px;
	}
	.message-wrap .message :global(a) {
		color: var(--color-text-link);
		text-decoration: underline;
	}

	.hide {
		display: none;
	}
</style>
