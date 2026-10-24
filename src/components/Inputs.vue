<script setup lang="ts">
import { computed, ref, type HTMLAttributes } from "vue";
import { Eye, EyeOff, Minus, Plus } from "@lucide/vue";
import { cn } from "../utils/cn";

// Attributes such as id, autocomplete or aria-* belong on the <input>, not on the wrapper used by password and number.
defineOptions({ inheritAttrs: false });

const props = withDefaults(
    defineProps<{
    type?: "text" | "password" | "email" | "number" | "date";
    size?: "sm" | "md" | "lg";
    min?: number;
    max?: number;
    step?: number;
    disabled?: boolean;
    placeholder?: string;
    name?: string;
    required?: boolean;
    class?: HTMLAttributes["class"];
    }>(),
    {
        type: "text",
        size: "md",
        step: 1,
        disabled: false,
        required: false,
    },
);

const model = defineModel<string | number>();

const sizes = {
    sm: "px-2.5 py-1.5 text-xs rounded-md",
    md: "px-3 py-2 text-sm rounded-lg",
    lg: "px-4 py-2.5 text-base rounded-lg",
};

// Room on the right for the password toggle or the number stepper, sized to the field.
const addons = {
    sm: { padding: "pr-8", width: "w-6", icon: "size-3", radius: "rounded-r-[5px]" },
    md: { padding: "pr-10", width: "w-7", icon: "size-3.5", radius: "rounded-r-[7px]" },
    lg: { padding: "pr-11", width: "w-8", icon: "size-4", radius: "rounded-r-[7px]" },
};

const showPassword = ref(false);
const hasAddon = computed(() => props.type === "password" || props.type === "number");

const inputClass = computed(() =>
    cn(
        "w-full border border-border bg-bg-surface text-text-primary placeholder:text-text-faint",
        sizes[props.size],
        hasAddon.value && addons[props.size].padding,
        "focus-visible:outline-2 focus-visible:outline-offset-2 focus-visible:outline-ring",
        "disabled:pointer-events-none disabled:opacity-50",
        hasAddon.value ? undefined : props.class,
    ),
);

const numberValue = computed(() => {
    const value = Number(model.value);

    return model.value === "" || model.value === undefined || !Number.isFinite(value) ? undefined : value;
});

function clamp(value: number) {
    return Math.min(props.max ?? Infinity, Math.max(props.min ?? -Infinity, value));
}

function stepBy(direction: 1 | -1) {
    const start = numberValue.value ?? (direction === 1 ? (props.min ?? 0) - props.step : (props.max ?? 0) + props.step);
    // Rounding keeps decimal steps such as 0.1 from drifting (0.1 + 0.2 = 0.30000000000000004).
    model.value = clamp(Math.round((start + direction * props.step) * 1e10) / 1e10);
}

const canIncrease = computed(() => props.max === undefined || numberValue.value === undefined || numberValue.value < props.max);
const canDecrease = computed(() => props.min === undefined || numberValue.value === undefined || numberValue.value > props.min);

const addonButtonClass = "flex flex-1 cursor-pointer items-center justify-center text-text-faint transition-colors hover:bg-bg-muted hover:text-text-primary active:bg-bg-muted-hover disabled:pointer-events-none disabled:opacity-40";

// "17102026" -> "17 / 10 / 2026"
function formatDate(value: string) {
    const digits = value.replace(/\D/g, "").slice(0, 8);

    return [digits.slice(0, 2), digits.slice(2, 4), digits.slice(4)]
        .filter(Boolean)
        .join(" / ");
}

function handleDateInput(event: Event) {
    const input = event.target as HTMLInputElement;

    input.value = formatDate(input.value);
    model.value = input.value;
}
</script>

<template>
    <input
        v-if="type === 'date'"
        v-bind="$attrs"
        :value="model"
        type="text"
        inputmode="numeric"
        maxlength="14"
        :disabled="disabled"
        :placeholder="placeholder ?? 'DD / MM / YYYY'"
        :name="name"
        :required="required"
        :class="inputClass"
        @input="handleDateInput"
    />

    <div v-else-if="type === 'password'" :class="cn('relative w-full', props.class)">
        <input
            v-bind="$attrs"
            v-model="model"
            :type="showPassword ? 'text' : 'password'"
            :disabled="disabled"
            :placeholder="placeholder"
            :name="name"
            :required="required"
            :class="inputClass"
        />
        <button
            type="button"
            :disabled="disabled"
            :aria-label="showPassword ? 'Hide password' : 'Show password'"
            :aria-pressed="showPassword"
            class="absolute inset-y-px right-px flex items-center justify-center text-text-faint transition-colors hover:text-text-primary focus-visible:outline-2 focus-visible:-outline-offset-2 focus-visible:outline-ring disabled:pointer-events-none disabled:opacity-40"
            :class="[addons[size].width, addons[size].radius]"
            @click="showPassword = !showPassword"
        >
            <EyeOff v-if="showPassword" :class="addons[size].icon" />
            <Eye v-else :class="addons[size].icon" />
        </button>
    </div>

    <div v-else-if="type === 'number'" :class="cn('relative w-full', props.class)">
        <input
            v-bind="$attrs"
            v-model="model"
            type="number"
            :min="min"
            :max="max"
            :step="step"
            :disabled="disabled"
            :placeholder="placeholder"
            :name="name"
            :required="required"
            :class="inputClass"
        />
        <div
            class="absolute inset-y-px right-px flex flex-col divide-y divide-border overflow-hidden border-l border-border"
            :class="[addons[size].width, addons[size].radius, { 'opacity-50': disabled }]"
        >
            <button
                type="button"
                tabindex="-1"
                aria-label="Increase"
                :disabled="disabled || !canIncrease"
                :class="addonButtonClass"
                @click="stepBy(1)"
            >
                <Plus :class="addons[size].icon" />
            </button>
            <button
                type="button"
                tabindex="-1"
                aria-label="Decrease"
                :disabled="disabled || !canDecrease"
                :class="addonButtonClass"
                @click="stepBy(-1)"
            >
                <Minus :class="addons[size].icon" />
            </button>
        </div>
    </div>

    <input
        v-else
        v-bind="$attrs"
        v-model="model"
        :type="type"
        :disabled="disabled"
        :placeholder="placeholder"
        :name="name"
        :required="required"
        :class="inputClass"
    />
</template>
