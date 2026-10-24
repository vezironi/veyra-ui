<script setup lang="ts">
    import { computed, type HTMLAttributes } from "vue";
    import { cn } from "../utils/cn";
    import Avatar from "./Avatar.vue";
    import Tooltip from "./Tooltip.vue";

    export interface AvatarGroupItem {
        name?: string;
        src?: string;
        alt?: string;
        fallback?: string;
    }

    const props = withDefaults(
        defineProps<{
            items: AvatarGroupItem[];
            max?: number;
            tooltip?: boolean;
            size?: "sm" | "md" | "lg";
            class?: HTMLAttributes["class"];
        }>(),
        {
            size: "md",
            tooltip: false,
        },
    );

    // Must match the Avatar sizes so the "+N" chip lines up with the avatars.
    const sizes = {
        sm: { overlap: "-space-x-1.5", chip: "size-8 text-xs" },
        md: { overlap: "-space-x-2", chip: "size-10 text-sm" },
        lg: { overlap: "-space-x-2.5", chip: "size-12 text-base" },
    };

    const chipClass = "relative inline-flex shrink-0 items-center justify-center rounded-full bg-bg-muted font-medium tabular-nums text-text-muted ring-2 ring-bg-base";

    const limit = computed(() =>
        props.max !== undefined && Number.isFinite(props.max) ? Math.max(1, Math.floor(props.max)) : props.items.length,
    );
    const visible = computed(() => props.items.slice(0, limit.value));
    const hidden = computed(() => props.items.length - visible.value.length);

    function itemLabel(item: AvatarGroupItem) {
        return item.name ?? item.alt ?? item.fallback ?? "";
    }

    // The +N chip lists the people it stands for.
    const hiddenLabel = computed(() => props.items.slice(limit.value).map(itemLabel).filter(Boolean).join(", "));
</script>

<template>
    <div role="group" :class="cn('flex items-center', sizes[size].overlap, props.class)">
        <template v-for="(item, index) in visible" :key="index">
            <Tooltip v-if="tooltip && itemLabel(item)" :content="itemLabel(item)">
                <Avatar
                    :src="item.src"
                    :alt="item.alt ?? item.name"
                    :fallback="item.fallback"
                    :size="size"
                    class="rounded-full ring-2 ring-bg-base"
                />
            </Tooltip>

            <Avatar
                v-else
                :src="item.src"
                :alt="item.alt ?? item.name"
                :fallback="item.fallback"
                :size="size"
                class="rounded-full ring-2 ring-bg-base"
            />
        </template>

        <template v-if="hidden > 0">
            <Tooltip v-if="tooltip && hiddenLabel" :content="hiddenLabel">
                <span :aria-label="`${hidden} more`" :class="cn(chipClass, sizes[size].chip)">+{{ hidden }}</span>
            </Tooltip>

            <span v-else :aria-label="`${hidden} more`" :class="cn(chipClass, sizes[size].chip)">+{{ hidden }}</span>
        </template>
    </div>
</template>
