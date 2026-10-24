<script setup lang="ts">
import { onBeforeUnmount, ref, type HTMLAttributes } from "vue";
import { cn } from "../utils/cn";

const props = withDefaults(
    defineProps<{
        content?: string;
        side?: "top" | "bottom" | "left" | "right";
        delay?: number;
        arrow?: boolean;
        class?: HTMLAttributes["class"];
    }>(),
    {
        side: "top",
        delay: 150,
        arrow: true,
    },
);

const triggerRef = ref<HTMLElement | null>(null);
const open = ref(false);

const tooltipStyle = ref<Record<string, string>>({});

let timer: ReturnType<typeof setTimeout> | undefined;

// A rotated square on the edge facing the trigger; its two outer borders form the point.
const arrowClasses = {
    top: "left-1/2 top-full -translate-x-1/2 -translate-y-1/2 border-b border-r",
    bottom: "bottom-full left-1/2 -translate-x-1/2 translate-y-1/2 border-l border-t",
    left: "left-full top-1/2 -translate-x-1/2 -translate-y-1/2 border-r border-t",
    right: "right-full top-1/2 translate-x-1/2 -translate-y-1/2 border-b border-l",
};

function updatePosition() {
    const trigger = triggerRef.value;

    if (!trigger) return;

    const rect = trigger.getBoundingClientRect();
    const gap = props.arrow ? 9 : 6;
    const centerX = `${rect.left + rect.width / 2}px`;
    const centerY = `${rect.top + rect.height / 2}px`;

    const positions = {
        top: { left: centerX, top: `${rect.top - gap}px`, transform: "translate(-50%, -100%)" },
        bottom: { left: centerX, top: `${rect.bottom + gap}px`, transform: "translateX(-50%)" },
        left: { left: `${rect.left - gap}px`, top: centerY, transform: "translate(-100%, -50%)" },
        right: { left: `${rect.right + gap}px`, top: centerY, transform: "translateY(-50%)" },
    };

    tooltipStyle.value = positions[props.side];
}

function show() {
    clearTimeout(timer);
    timer = setTimeout(() => {
        updatePosition();
        open.value = true;
    }, props.delay);
}

function hide() {
    clearTimeout(timer);
    open.value = false;
}

onBeforeUnmount(() => clearTimeout(timer));
</script>

<template>
    <span
        ref="triggerRef"
        class="inline-flex w-fit max-w-full"
        @mouseenter="show"
        @mouseleave="hide"
        @focusin="show"
        @focusout="hide"
    >
        <slot />

        <Teleport to="body">
            <div
                v-if="open"
                role="tooltip"
                :class="cn('pointer-events-none fixed z-[9999] whitespace-nowrap rounded-md border border-border bg-bg-surface px-2 py-1 text-xs text-text-primary', props.class)"
                :style="tooltipStyle"
            >
                <slot name="content">{{ content }}</slot>
                <span
                    v-if="arrow"
                    aria-hidden="true"
                    class="absolute size-2 rotate-45 border-border bg-bg-surface"
                    :class="arrowClasses[side]"
                />
            </div>
        </Teleport>
    </span>
</template>
