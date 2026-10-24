<script setup lang="ts">
import { computed, type Component, type HTMLAttributes } from "vue";
import { Minus, TrendingDown, TrendingUp } from "@lucide/vue";
import { cn } from "../utils/cn";
import AnimatedNumber from "./AnimatedNumber.vue";

const props = withDefaults(
    defineProps<{
        icon?: Component | string;
        label: string;
        value: string | number;
        suffix?: string;
        change?: number;
        changeLabel?: string;
        color?: string;
        class?: HTMLAttributes["class"];
    }>(),
    {
        icon: undefined,
        label: '',
        value: '',
        suffix: undefined,
        change: undefined,
        changeLabel: undefined,
        color: undefined,
    },
);

// change is a percentage: 12 -> "+12%", -4.5 -> "-4.5%".
const trend = computed(() => {
    const change = props.change;

    if (change === undefined || !Number.isFinite(change)) return undefined;
    if (change > 0) return { icon: TrendingUp, text: `+${change}%`, class: "bg-success/10 text-success" };
    if (change < 0) return { icon: TrendingDown, text: `${change}%`, class: "bg-danger/10 text-danger" };

    return { icon: Minus, text: "0%", class: "bg-bg-muted text-text-muted" };
});
</script>

<template>
    <div :class="cn('w-full flex flex-col p-4 rounded-2xl border border-border bg-bg-surface min-w-0', props.class)">
        <div class="flex items-start justify-between gap-x-3">
            <p class="truncate text-sm text-text-muted">{{ label }}</p>
            <span
                v-if="icon"
                class="flex size-8 shrink-0 items-center justify-center rounded-lg bg-bg-muted text-text-muted"
            >
                <component :is="icon" class="size-4" />
            </span>
        </div>

        <div class="mt-2 flex min-w-0 items-baseline gap-x-1.5">
            <p
                class="flex text-2xl font-bold leading-none tabular-nums whitespace-nowrap"
                :style="{ color: color || 'var(--color-text-primary)' }"
            >
                <AnimatedNumber :value="value" />
            </p>
            <span v-if="suffix" class="truncate text-sm text-text-faint">{{ suffix }}</span>
        </div>

        <div v-if="trend || changeLabel" class="mt-2 flex items-center gap-x-1.5 text-xs">
            <span
                v-if="trend"
                class="inline-flex items-center gap-x-1 rounded-full px-1.5 py-0.5 font-medium tabular-nums"
                :class="trend.class"
            >
                <component :is="trend.icon" class="size-3" />
                {{ trend.text }}
            </span>
            <span v-if="changeLabel" class="truncate text-text-faint">{{ changeLabel }}</span>
        </div>
    </div>
</template>
