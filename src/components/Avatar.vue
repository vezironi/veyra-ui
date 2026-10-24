<script setup lang="ts">
    import type { HTMLAttributes } from "vue";
    import { cn } from "../utils/cn";
    import Tooltip from "./Tooltip.vue";

    const props = withDefaults(
        defineProps<{
            src?: string;
            alt?: string;
            fallback?: string;
            size?: "sm" | "md" | "lg";
            badge?: boolean;
            badgeVariant?: "active" | "inactive" | "busy";
            badgeLabel?: string;
            badgeTooltip?: boolean;
            class?: HTMLAttributes["class"];
        }>(),
        {
            alt: "",
            fallback: "?",
            size: "md",
            badge: false,
            badgeVariant: "active",
            badgeTooltip: true,
        },
    );

    const sizes = {
        sm: "size-8 text-xs",
        md: "size-10 text-sm",
        lg: "size-12 text-base",
    };

    const badgeVariants = {
        active: "bg-success",
        inactive: "bg-zinc-500",
        busy: "bg-danger",
    };

    const badgeLabels = {
        active: "Online",
        inactive: "Offline",
        busy: "Do not disturb",
    };
</script>

<template>
    <span :class="cn('relative inline-flex shrink-0', sizes[size], props.class)">
        <img
            v-if="src"
            :src="src"
            :alt="alt"
            draggable="false"
            class="size-full rounded-full object-cover"
            @error="($event.target as HTMLImageElement).style.display = 'none'"
        />

        <span
            v-if="!src"
            class="flex size-full items-center justify-center rounded-full bg-bg-elevated font-medium text-text-primary"
        >
            {{ fallback }}
        </span>

        <span v-if="badge" class="absolute bottom-0 right-0 flex">
            <Tooltip v-if="badgeTooltip" :content="badgeLabel ?? badgeLabels[badgeVariant]">
                <span
                    role="img"
                    :aria-label="badgeLabel ?? badgeLabels[badgeVariant]"
                    class="size-3 rounded-full ring-2 ring-bg-base"
                    :class="badgeVariants[badgeVariant]"
                />
            </Tooltip>
            <span
                v-else
                role="img"
                :aria-label="badgeLabel ?? badgeLabels[badgeVariant]"
                class="size-3 rounded-full ring-2 ring-bg-base"
                :class="badgeVariants[badgeVariant]"
            />
        </span>
    </span>
</template>
