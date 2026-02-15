<script>
	// Svelte 5 — runes mode

	let {
		value = $bindable(0), // total seconds
		showHours = false
	} = $props();

	let inputEl = $state();
	let activeSegment = $state(0);
	let segments = $state([]);
	let digitBuffer = $state('');

	/* ---------- Time helpers ---------- */

	function normalize(total) {
		return Math.max(0, Math.floor(total));
	}

	function toParts(totalSeconds) {
		totalSeconds = normalize(totalSeconds);

		return {
			hours: Math.floor(totalSeconds / 3600),
			minutes: Math.floor((totalSeconds % 3600) / 60),
			seconds: totalSeconds % 60
		};
	}

	function format(totalSeconds) {
		const { hours, minutes, seconds } = toParts(totalSeconds);

		const mm = String(minutes).padStart(2, '0');
		const ss = String(seconds).padStart(2, '0');

		if (showHours || hours > 0) {
			const hh = String(hours).padStart(2, '0');
			return `${hh}:${mm}:${ss}`;
		}

		return `${mm}:${ss}`;
	}

	function buildSegments(text) {
		const parts = text.split(':');
		let cursor = 0;

		return parts.map((part, i) => {
			const start = cursor;
			const end = cursor + part.length;
			cursor = end + 1;
			return { index: i, start, end };
		});
	}

	$effect(() => {
		segments = buildSegments(format(value));
	});

	function hasHours() {
		const { hours } = toParts(value);
		return showHours || hours > 0;
	}

	/* ---------- Segment selection ---------- */

	function selectSegment(index) {
		const seg = segments[index];
		if (!seg || !inputEl) return;

		activeSegment = index;
		digitBuffer = '';
		inputEl.setSelectionRange(seg.start, seg.end);
	}

	function moveSegment(delta) {
		const next =
			(activeSegment + delta + segments.length) % segments.length;
		selectSegment(next);
	}

	/* ---------- Smart rollover logic ---------- */

	function adjustSegment(index, delta) {
		let total = value;
		const hoursVisible = hasHours();

		if (hoursVisible) {
			if (index === 0) total += delta * 3600;
			if (index === 1) total += delta * 60;
			if (index === 2) total += delta;
		} else {
			if (index === 0) total += delta * 60;
			if (index === 1) total += delta;
		}

		value = normalize(total);

		queueMicrotask(() => selectSegment(index));
	}

	function replaceSegment(index, newVal) {
		let { hours, minutes, seconds } = toParts(value);
		const hoursVisible = hasHours();

		if (hoursVisible) {
			if (index === 0) hours = newVal;
			if (index === 1) minutes = newVal;
			if (index === 2) seconds = newVal;
		} else {
			if (index === 0) minutes = newVal;
			if (index === 1) seconds = newVal;
		}

		// Smart rollover by reconstructing total seconds
		value = normalize(hours * 3600 + minutes * 60 + seconds);

		queueMicrotask(() => selectSegment(index));
	}

	function getSegmentValue(index) {
		const { hours, minutes, seconds } = toParts(value);
		const hoursVisible = hasHours();

		if (hoursVisible) {
			if (index === 0) return hours;
			if (index === 1) return minutes;
			if (index === 2) return seconds;
		} else {
			if (index === 0) return minutes;
			if (index === 1) return seconds;
		}
		return 0;
	}

	/* ---------- Events ---------- */

	function handleFocus(e) {
		// If focus came from mouse click,
		// selectionStart is already set.
		if (e.relatedTarget === null && inputEl.selectionStart !== null) {
			handleClick(e);
			return;
		}

		selectSegment(0);
	}

	function handleClick(e) {
		const pos = inputEl.selectionStart;

		for (const seg of segments) {
			if (pos >= seg.start && pos <= seg.end) {
				selectSegment(seg.index);
				break;
			}
		}
	}

	function handleKeydown(e) {
		const segmentCount = segments.length;

		/* --- Tab behavior --- */
		if (e.key === 'Tab') {
			if (!e.shiftKey && activeSegment === segmentCount - 1) return;
			if (e.shiftKey && activeSegment === 0) return;

			e.preventDefault();
			moveSegment(e.shiftKey ? -1 : 1);
			return;
		}

		/* --- Arrow navigation --- */
		if (e.key === 'ArrowRight') {
			e.preventDefault();
			moveSegment(1);
			return;
		}

		if (e.key === 'ArrowLeft') {
			e.preventDefault();
			moveSegment(-1);
			return;
		}

		/* --- Increment / decrement with rollover --- */
		if (e.key === 'ArrowUp') {
			e.preventDefault();
			adjustSegment(activeSegment, 1);
			return;
		}

		if (e.key === 'ArrowDown') {
			e.preventDefault();
			adjustSegment(activeSegment, -1);
			return;
		}

		/* --- Numeric input with rollover --- */
		if (/^\d$/.test(e.key)) {
			e.preventDefault();

			digitBuffer = (digitBuffer + e.key).slice(-2);

			replaceSegment(activeSegment, Number(digitBuffer));

			if (digitBuffer.length === 2) {
				if (activeSegment < segmentCount - 1) {
					queueMicrotask(() => moveSegment(1));
				}
				digitBuffer = '';
			}
		}
	}
</script>

<input
	bind:this={inputEl}
	value={format(value)}
	inputmode="numeric"
	on:focus={handleFocus}
	on:click={handleClick}
	on:keydown={handleKeydown}
	style="width: 8ch; font-variant-numeric: tabular-nums;"
/>
