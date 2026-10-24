<script setup lang="ts">
import { computed, ref, watch, type HTMLAttributes } from "vue";
import { cn } from "../utils/cn";

// Odometer-style value: only the digits that change roll, the rest stay put ("14" -> "18" rolls just the 4).
const props = defineProps<{
    value: string | number;
    class?: HTMLAttributes["class"];
}>();

const text = computed(() => String(props.value));
const direction = ref<"up" | "down">("up");

function toNumber(value: string) {
    return Number(value.replace(/[^\d.-]/g, ""));
}

watch(text, (next, previous) => {
    direction.value = toNumber(next) < toNumber(previous) ? "down" : "up";
});

// Keyed from the right so ones stay aligned with ones when the number gains a digit.
const columns = computed(() => {
    const chars = [...text.value];

    return chars.map((char, index) => ({ key: chars.length - index, char, digit: /\d/.test(char) }));
});
</script>

<template>
    <span :class="cn('inline-flex tabular-nums', props.class)">
        <span class="sr-only">{{ text }}</span>
        <span aria-hidden="true" class="inline-flex">
            <span v-for="column in columns" :key="column.key" class="relative inline-flex overflow-hidden">
                <Transition v-if="column.digit" :name="`roll-${direction}`">
                    <span :key="column.char" class="inline-block">{{ column.char }}</span>
                </Transition>
                <span v-else>{{ column.char }}</span>
            </span>
        </span>
    </span>
</template>

<style scoped>
.roll-up-enter-active,
.roll-up-leave-active,
.roll-down-enter-active,
.roll-down-leave-active {
    transition:
        transform 0.35s cubic-bezier(0.2, 0.8, 0.2, 1),
        opacity 0.35s ease;
}

/* The outgoing digit leaves the flow so the incoming one keeps the column width. */
.roll-up-leave-active,
.roll-down-leave-active {
    position: absolute;
    inset: 0;
}

.roll-up-enter-from,
.roll-down-leave-to {
    transform: translateY(100%);
    opacity: 0;
}

.roll-up-leave-to,
.roll-down-enter-from {
    transform: translateY(-100%);
    opacity: 0;
}

@media (prefers-reduced-motion: reduce) {
    .roll-up-enter-active,
    .roll-up-leave-active,
    .roll-down-enter-active,
    .roll-down-leave-active {
        transition: none;
    }
}
</style>
