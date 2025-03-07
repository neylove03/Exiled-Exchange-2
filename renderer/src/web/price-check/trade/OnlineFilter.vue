<template>
  <ui-popover
    :delay="[80, null]"
    placement="bottom-start"
    boundary="#price-window"
  >
    <template #target>
      <button
        class="rounded mr-1 px-2 truncate"
        :class="showWarning() ? 'text-orange-500' : 'text-gray-500'"
      >
        <span
          ><i class="fas fa-history"></i>
          {{ t(filters.trade.offline ? "Offline" : "Online") }}</span
        >
        <span v-if="showLeagueName()">, {{ filters.trade.league }}</span>
      </button>
    </template>
    <template #content>
      <div class="flex gap-x-8 p-2 bg-gray-800 text-gray-400">
        <div class="flex flex-col gap-y-1">
          <div class="mb-1">
            <ui-toggle
              :modelValue="filters.trade.offline"
              @update:modelValue="onOfflineUpdate"
              >{{ t(":offline_toggle") }}</ui-toggle
            >
          </div>
          <template v-if="byTime">
            <ui-radio v-model="filters.trade.listed" :value="undefined">{{
              t(":listed_any_time")
            }}</ui-radio>
            <ui-radio v-model="filters.trade.listed" value="1day">{{
              t(":listed_1day")
            }}</ui-radio>
            <ui-radio v-model="filters.trade.listed" value="3days">{{
              t(":listed_3days")
            }}</ui-radio>
            <ui-radio v-model="filters.trade.listed" value="1week">{{
              t(":listed_1week")
            }}</ui-radio>
            <ui-radio v-model="filters.trade.listed" value="2weeks">{{
              t(":listed_2weeks")
            }}</ui-radio>
            <ui-radio v-model="filters.trade.listed" value="1month">{{
              t(":listed_1month")
            }}</ui-radio>
            <ui-radio v-model="filters.trade.listed" value="2months">{{
              t(":listed_2months")
            }}</ui-radio>
          </template>
        </div>
        <div class="flex flex-col gap-y-1">
          <div class="mb-1">
            <ui-toggle
              :class="{ invisible: filters.trade.offline }"
              v-model="filters.trade.onlineInLeague"
              >{{ t(":in_league_toggle") }}</ui-toggle
            >
          </div>
          <ui-radio
            v-for="league of tradeLeagues"
            :key="league.id"
            v-model="filters.trade.league"
            :value="league.id"
            >{{ league.id }}</ui-radio
          >
          <template v-if="byTime">
            <ui-radio
              class="mt-3"
              v-model="filters.trade.currency"
              :value="undefined"
              >{{ t(":currency_any") }}</ui-radio
            >
            <ui-radio v-model="filters.trade.currency" value="exalted">{{
              t(":currency_only_exalted")
            }}</ui-radio>
            <!-- <ui-radio v-model="filters.trade.currency" value="chaos">{{
              t(":currency_only_chaos")
            }}</ui-radio> -->
            <ui-radio v-model="filters.trade.currency" value="divine">{{
              t(":currency_only_div")
            }}</ui-radio>
            <ui-radio v-model="filters.trade.currency" value="exalted_divine">{{
              t(":currency_exalted_div")
            }}</ui-radio>
          </template>
        </div>
        <div class="flex flex-col gap-y-1">
          <template v-if="currencyRatio">
            <ui-popover
              tag-name="div"
              class="flex"
              placement="top"
              boundary="#price-window"
            >
              <template #target>
                <div>{{ localCurrencyRatio }} ex : 1 div</div>
              </template>
              <template #content>
                <div style="max-width: 18.5rem">
                  {{ t(":ratio_tooltip") }}
                </div>
              </template>
            </ui-popover>
            <input
              id="currency-ratio"
              type="range"
              v-model="localCurrencyRatio"
              min="0"
              max="300"
              step="1"
              @mouseup="updateCurrencyRatio"
              @touchend="updateCurrencyRatio"
            />
          </template>
        </div>
      </div>
    </template>
  </ui-popover>
</template>

<script lang="ts">
import { defineComponent, PropType, ref } from "vue";
import { useI18nNs } from "@/web/i18n";
import UiRadio from "@/web/ui/UiRadio.vue";
import UiToggle from "@/web/ui/UiToggle.vue";
import UiPopover from "@/web/ui/Popover.vue";
import type { ItemFilters } from "../filters/interfaces";
import { useLeagues } from "@/web/background/Leagues";
import { CURRENCY_RATIO } from "../filters/create-item-filters";

export default defineComponent({
  components: { UiRadio, UiToggle, UiPopover },
  props: {
    filters: {
      type: Object as PropType<ItemFilters>,
      required: true,
    },
    byTime: {
      type: Boolean,
      default: false,
    },
    currencyRatio: {
      type: Boolean,
      default: false,
    },
  },
  setup(props) {
    const leagues = useLeagues();
    const { t } = useI18nNs("online_filter");

    const localCurrencyRatio = ref(props.filters.trade.currencyRatio);

    const updateCurrencyRatio = () => {
      props.filters.trade.currencyRatio = localCurrencyRatio.value;
    };

    if (!props.filters.trade.currencyRatio) {
      updateCurrencyRatio();
    }

    return {
      t,
      tradeLeagues: leagues.list,
      localCurrencyRatio,
      updateCurrencyRatio,
      showLeagueName: () =>
        leagues.selectedId.value !== props.filters.trade.league,
      showWarning: () =>
        Boolean(
          (props.filters.trade.listed &&
            ["1day", "3days", "1week"].includes(props.filters.trade.listed)) ||
            props.filters.trade.currency ||
            CURRENCY_RATIO !== props.filters.trade.currencyRatio,
        ),
      onOfflineUpdate(offline: boolean) {
        const { filters } = props;
        filters.trade.offline = offline;
        if (props.byTime) {
          filters.trade.listed = offline ? "2months" : undefined;
        }
      },
    };
  },
});
</script>
