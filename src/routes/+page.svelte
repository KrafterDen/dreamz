<script lang="ts">
	import { tick } from 'svelte';
	import logo from '../design_items/logo.png';
	import plusIcon from '../design_items/plus.png';
	import arrowIcon from '../design_items/arrow.png';
	import happyIcon from '../design_items/happy.png';
	import sadIcon from '../design_items/sad.png';

	const categories = ['dreams', 'places', 'characters'] as const;
	const selectedCategory = 'dreams';

	type Mood = 'happy' | 'sad';

	type Dream = {
		title: string;
		content: string;
		time: string;
		mood?: Mood | null;
	};

	type DreamGroup = {
		displayDate: string;
		isoDate: string;
		entries: Dream[];
	};

	const initialGroups: DreamGroup[] = [
		{
			displayDate: '26 june',
			isoDate: '2024-06-26',
			entries: [
				{
					title: 'Оргія',
					content:
						'Я смачню, кожную в полку, ебал коровок і они постоянно отбивались от меня, но я не сдавался...',
					time: '03:24'
				},
				{
					title: 'Мяса',
					content:
						'МясаМясаМясаМясаМясаМясаМясаМясаМясаМясаМясаМясаМясаМясаМясаМясаМясаМясаМяса...',
					time: '05:10'
				},
				{
					title: 'Человек паук',
					content:
						'я в москве с бандой человеков пауков и мы отбиваем булочную и потом успешно возвращаемся...',
					time: '06:52'
				}
			]
		},
		{
			displayDate: '25 june',
			isoDate: '2024-06-25',
			entries: [
				{
					title: 'Солана по 380',
					content: 'как солана поднялась до 380, но мне показалось 380 было 180 баксов',
					time: '01:03'
				},
				{
					title: 'Бассейн пива',
					content:
						'я с друзьями на пирогах купаюсь в бассейне с пивом и ищу деревянную лесенку наверх...',
					time: '02:40'
				}
			]
		}
	];

let dreamGroups: DreamGroup[] = initialGroups;
let currentScreen: 'list' | 'create' = 'list';
let titleValue = '';
let dreamValue = '';
let selectedMood: Mood | null = null;

let timelineRef: HTMLElement | null = null;
let titleInputRef: HTMLTextAreaElement | null = null;
let dreamInputRef: HTMLTextAreaElement | null = null;

	const truncate = (value: string, max = 100) =>
		value.length > max ? `${value.slice(0, max)}…` : value;

	const formatDisplayDate = (date: Date) =>
		new Intl.DateTimeFormat('en-GB', { day: 'numeric', month: 'long' })
			.format(date)
			.toLowerCase();

	const formatTime = (date: Date) =>
		new Intl.DateTimeFormat('en-GB', {
			hour: '2-digit',
			minute: '2-digit',
			hour12: false
		}).format(date);

	function addDreamToGroups(groups: DreamGroup[], dream: Dream, date: Date) {
		const isoDate = date.toISOString().slice(0, 10);
		const displayDate = formatDisplayDate(date);

		const next = groups.map((group) => ({ ...group, entries: [...group.entries] }));
		const existing = next.find((group) => group.isoDate === isoDate);

		if (existing) {
			existing.entries = [dream, ...existing.entries];
		} else {
			next.push({
				displayDate,
				isoDate,
				entries: [dream]
			});
		}

		next.sort((a, b) => (a.isoDate > b.isoDate ? -1 : 1));
		return next;
	}

	function resetForm() {
		titleValue = '';
		dreamValue = '';
		selectedMood = null;
	}

async function openCreate() {
	currentScreen = 'create';
	await tick();
	titleInputRef?.dispatchEvent(new Event('input'));
	dreamInputRef?.dispatchEvent(new Event('input'));
}

	function closeCreate() {
		resetForm();
		currentScreen = 'list';
	}

	async function submitDream() {
		const trimmedTitle = titleValue.trim();
		const trimmedDream = dreamValue.trim();

		if (!trimmedTitle || !trimmedDream) {
			return;
		}

		const now = new Date();
		const newDream: Dream = {
			title: trimmedTitle,
			content: trimmedDream,
			time: formatTime(now),
			mood: selectedMood
		};

		dreamGroups = addDreamToGroups(dreamGroups, newDream, now);
		await tick();
		timelineRef?.scrollTo({ top: 0 });

		resetForm();
		currentScreen = 'list';
	}

	function toggleMood(mood: Mood) {
		selectedMood = selectedMood === mood ? null : mood;
	}

	function pixelPress(node: HTMLElement) {
		const press = () => node.classList.add('pressed');
		const release = () => node.classList.remove('pressed');

		node.addEventListener('pointerdown', press);
		node.addEventListener('pointerup', release);
		node.addEventListener('pointerleave', release);
		node.addEventListener('pointercancel', release);

		return {
			destroy() {
				node.removeEventListener('pointerdown', press);
				node.removeEventListener('pointerup', release);
				node.removeEventListener('pointerleave', release);
				node.removeEventListener('pointercancel', release);
			}
		};
	}

	function autosize(node: HTMLTextAreaElement, value: string) {
		const minHeight = Number(node.dataset.minHeight) || 96;
		const resize = () => {
			node.style.height = 'auto';
			node.style.height = `${Math.max(node.scrollHeight, minHeight)}px`;
		};

		resize();
		node.addEventListener('input', resize);

		return {
			update(newValue: string) {
				if (newValue !== value) {
					value = newValue;
					requestAnimationFrame(resize);
				}
			},
			destroy() {
				node.removeEventListener('input', resize);
			}
		};
	}
</script>

<div class="app" data-screen={currentScreen}>
	{#if currentScreen === 'list'}
		<div class="list-screen">
			<header class="top-bar">
				<div class="branding">
					<img class="logo" src={logo} alt="Dreamzz logo" />
					<span class="wordmark">dreamz</span>
				</div>
			</header>

			<nav class="category-tabs" aria-label="Sections">
				{#each categories as category}
					<button type="button" class:selected={category === selectedCategory}>
						{category}
					</button>
				{/each}
			</nav>

			<main class="timeline" bind:this={timelineRef}>
				{#each dreamGroups as group (group.isoDate)}
					<section class="date-group" aria-labelledby={group.isoDate}>
						<div class="date-tag" id={group.isoDate}>
							{group.displayDate}
						</div>
						<ul class="dream-items">
							{#each group.entries as dream (dream.title + dream.time)}
								<li class="dream-card">
									<div class="card-header">
										<div class="dream-header">{dream.title}</div>
										<time datetime={`${group.isoDate}T${dream.time}`}>{dream.time}</time>
									</div>
									<p>{truncate(dream.content)}</p>
								</li>
							{/each}
						</ul>
					</section>
				{/each}
			</main>

            <button
                class="floating-action pixel-press"
                type="button"
                aria-label="Add dream"
                style={`background-image: url(${plusIcon});`}
                use:pixelPress
                on:pointerup|once={() => setTimeout(() => openCreate(), 160)}
            />
		</div>
	{:else}
		<section class="create-screen">
			<div class="create-top">
				<button
					class="icon-button arrow pixel-press"
					type="button"
					aria-label="Back to dream list"
					style={`background-image: url(${arrowIcon});`}
					use:pixelPress
					on:pointerup|once={() => setTimeout(() => closeCreate(), 160)}
				/>
				<span class="form-heading">title</span>
				<span class="form-heading spacer" aria-hidden="true"></span>
			</div>

		<textarea
			class="text-input title-input"
			placeholder="enter your title"
			bind:value={titleValue}
			class:filled={titleValue.trim().length > 0}
			use:autosize={titleValue}
			bind:this={titleInputRef}
			data-min-height="96"
		/>

		<span class="form-heading secondary">dream</span>

		<textarea
			class="text-input dream-input"
			placeholder="enter your dream"
			bind:value={dreamValue}
			class:filled={dreamValue.trim().length > 0}
			use:autosize={dreamValue}
			bind:this={dreamInputRef}
			data-min-height="144"
		/>

			<div class="actions-row">
				<button
					class="mood-button pixel-press"
					type="button"
					aria-label="Mark as sad dream"
					aria-pressed={selectedMood === 'sad'}
					style={`background-image: url(${sadIcon});`}
					use:pixelPress
					on:click={() => toggleMood('sad')}
					class:selected={selectedMood === 'sad'}
				/>

				<button
					class="submit-button pixel-press"
					type="button"
					use:pixelPress
					on:click={submitDream}
				>
					SUBMIT
				</button>

				<button
					class="mood-button pixel-press"
					type="button"
					aria-label="Mark as happy dream"
					aria-pressed={selectedMood === 'happy'}
					style={`background-image: url(${happyIcon});`}
					use:pixelPress
					on:click={() => toggleMood('happy')}
					class:selected={selectedMood === 'happy'}
				/>
			</div>
		</section>
	{/if}
</div>

<style>
	:global(html, body) {
		margin: 0;
		height: 100%;
		overflow: hidden;
	}

	:global(body) {
		background: #050505;
		color: #f5f5f5;
		font-family: 'Handjet', 'IBM Plex Mono', 'Courier New';
	}

	:global(*) {
		box-sizing: border-box;
	}

	.app {
		position: relative;
		height: 100vh;
		width: 100vw;
		background: linear-gradient(180deg, #090909 0%, #050505 100%);
		display: flex;
		flex-direction: column;
		box-shadow: 0 0 0 1px #1e1e1e, 0 0 30px rgba(0, 0, 0, 0.65);
		overflow: hidden;
	}

	.list-screen {
		display: flex;
		flex-direction: column;
		height: 100%;
	}

	.top-bar {
		display: flex;
		align-items: center;
		padding: 0.75rem 1rem;
		border: 1px solid #2c2c2c;
		background: #0d0d0d;
		box-shadow: inset 0 0 0 1px #1a1a1a;
	}

	.branding {
		display: flex;
		align-items: center;
		gap: 0.75rem;
	}

	.logo {
		width: 36px;
		height: 36px;
		object-fit: contain;
		filter: grayscale(1);
	}

	.wordmark {
		font-size: 2.25rem;
		letter-spacing: 0.08em;
		text-transform: lowercase;
		color: #454545;
		text-shadow: 0 0 4px rgba(255, 255, 255, 0.18);
	}

	.category-tabs {
		display: grid;
		grid-template-columns: repeat(3, 1fr);
		border: 1px solid #2c2c2c;
		background: #0b0b0b;
		box-shadow: inset 0 0 0 1px #161616;
	}

	.category-tabs button {
		font-family: Handjet;
		padding: 0.75rem 0.5rem;
		border: none;
		background: transparent;
		color: #999999;
		font-size: 1.5rem;
		letter-spacing: 0.05em;
		display: flex;
		align-items: center;
		justify-content: center;
		position: relative;
	}

	.category-tabs button::after {
		content: '';
		position: absolute;
		top: 20%;
		bottom: 20%;
		right: 0;
		width: 1px;
		background: #2c2c2c;
	}

	.category-tabs button:last-child::after {
		display: none;
	}

	.category-tabs button.selected {
		color: #f5f5f5;
		background: #141414;
		box-shadow: inset 0 0 0 1px #242424;
	}

	.category-tabs button:focus-visible {
		outline: 2px dashed #f5f5f5;
		outline-offset: -4px;
	}

	.timeline {
		display: flex;
		flex-direction: column;
		gap: 0;
		flex: 1;
		overflow-y: auto;
		padding: 0 0 5.5rem;
		-webkit-overflow-scrolling: touch;
		scrollbar-width: none;
		-ms-overflow-style: none;
	}

	.timeline::-webkit-scrollbar {
		display: none;
	}

	.date-tag {
		display: flex;
		align-items: center;
		justify-content: center;
		padding: 0.5rem 0.75rem;
		background: #0d0d0d;
		color: #454545;
		border: 1px solid #2b2b2b;
		font-size: 1.5rem;
		letter-spacing: 0.12em;
		text-transform: lowercase;
		box-shadow: inset 0 0 0 1px #171717;
	}

	.dream-items {
		list-style: none;
		margin: 0;
		padding: 0;
		display: flex;
		flex-direction: column;
	}

	.dream-card {
		padding: 1rem;
		background: #0b0b0b;
		border: 1px solid #303030;
		box-shadow: inset 0 0 0 1px #151515;
		display: flex;
		flex-direction: column;
		gap: 0.5rem;
	}

	.card-header {
		display: flex;
		justify-content: space-between;
		align-items: baseline;
		gap: 1rem;
	}

	.dream-header {
		font-size: 2rem;
		letter-spacing: 0.04em;
		color: #c5c5c5;
	}

	time {
		font-size: 0.85rem;
		color: #8a8a8a;
		letter-spacing: 0.08em;
	}

	p {
		margin: 0;
		color: #454545;
		font-size: 1rem;
		line-height: 1.4;
		letter-spacing: 0.02em;
		overflow-wrap: anywhere;
		word-break: break-word;
		white-space: normal;
		display: -webkit-box;
		-webkit-line-clamp: 3;
		-webkit-box-orient: vertical;
	}

	.pixel-press {
		transition: transform 40ms steps(1, end);
	}

	.pixel-press:active,
	.pixel-press.pressed {
		transform: scale(0.85);
	}

	@media (prefers-reduced-motion: reduce) {
		.pixel-press {
			transition: none;
		}
	}

	.floating-action {
		position: fixed;
		bottom: calc(env(safe-area-inset-bottom) + 1rem);
		right: calc(env(safe-area-inset-right) + 1rem);
    	width: 96px;
    	height: 96px;
		border: none;
		padding: 0;
		background-color: transparent;
		background-repeat: no-repeat;
		background-position: center;
		background-size: contain;
		image-rendering: pixelated;
		z-index: 10;
	}

	.floating-action:focus-visible {
		outline: 2px dashed #f5f5f5;
		outline-offset: 4px;
	}

    .create-screen {
        display: flex;
        flex-direction: column;
        /* Keep tiny 2px side inset to avoid edge clipping */
        padding: 1rem 2px 2rem;
        gap: 1rem;
        height: 100%;
        overflow-y: auto;
    }

    .create-top {
        display: grid;
        grid-template-columns: 48px 1fr 48px;
        align-items: center;
        padding: 0 0; /* align to screen edge */
    }

	.icon-button {
		width: 48px;
		height: 48px;
		background-color: transparent;
		background-repeat: no-repeat;
		background-position: center;
		background-size: contain;
		border: none;
		padding: 0;
		image-rendering: pixelated;
	}

	.icon-button:focus-visible {
		outline: 2px dashed #f5f5f5;
		outline-offset: 4px;
	}

	.form-heading {
		text-align: center;
		font-size: 2rem;
		color: #999999;
		text-transform: lowercase;
		letter-spacing: 0.05em;
	}

	.form-heading.spacer {
		opacity: 0;
	}

    .form-heading + .text-input {
        margin-top: 0.25rem; /* tighten heading to its input */
    }

    .form-heading.secondary {
        padding-top: 0.35rem; /* smaller gap above second header */
    }

    .text-input {
        width: 100%;
        border: 1px solid #303030;
        box-shadow: inset 0 0 0 1px #151515;
        background: #0b0b0b;
        color: #454545;
        font-family: Handjet;
        font-size: 2rem;
        line-height: 1.4;
        padding: 0.75rem;
        min-height: 64px;
        letter-spacing: 0.04em;
        resize: none;
        /* extend to full screen width while the screen keeps 2px inset */
        margin-left: -2px;
        margin-right: -2px;
    }

	.text-input.title-input {
		min-height: 96px;
		text-align: center;
	}

	.text-input::placeholder {
		color: #454545;
		font-weight: 300;
	}

	.text-input.title-input.filled {
		color: #c5c5c5;
		font-weight: 400;
	}

	.text-input.dream-input {
		min-height: 144px;
		text-align: center;
	}

	.text-input.dream-input.filled {
		color: #c5c5c5;
		font-size: 1.25rem;
		font-weight: 400;
		line-height: 1.6;
	}

	.text-input:focus-visible {
		outline: 2px dashed #f5f5f5;
		outline-offset: 4px;
	}

	.actions-row {
		display: grid;
		grid-template-columns: 64px 1fr 64px;
		align-items: center;
		gap: 1rem;
		padding-top: 1rem;
	}

	.mood-button {
		width: 64px;
		height: 64px;
		border: none;
		background-repeat: no-repeat;
		background-position: center;
		background-size: contain;
		background-color: transparent;
		image-rendering: pixelated;
	}

	.mood-button.selected {
		filter: brightness(1.2);
	}

	.mood-button:focus-visible {
		outline: 2px dashed #f5f5f5;
		outline-offset: 4px;
	}

	.submit-button {
		font-family: Handjet;
		border: none;
		background: #dadada;
		color: #222222;
		font-size: 2rem;
		padding: 0.75rem 1rem;
		letter-spacing: 0.08em;
		text-transform: uppercase;
		box-shadow: inset 0 0 0 1px rgba(0, 0, 0, 0.35);
	}

	.submit-button:focus-visible {
		outline: 2px dashed #f5f5f5;
		outline-offset: 4px;
	}

	@font-face {
		font-family: 'Handjet';
		src: url('../design_items/fonts/Handjet-VariableFont_ELGR,ELSH,wght.ttf') format('truetype');
		font-weight: 100 900;
		font-style: normal;
		font-display: swap;
	}

	@media (max-width: 420px) {
		.create-screen {
			padding: 1rem 1.25rem 2rem;
		}

		.create-top {
			grid-template-columns: 42px 1fr 42px;
		}

		.icon-button {
			width: 36px;
			height: 36px;
		}

		.actions-row {
			grid-template-columns: 56px 1fr 56px;
		}

		.mood-button {
			width: 56px;
			height: 56px;
		}

    .floating-action {
      width: 96px;
      height: 96px;
    }
	}
</style>
