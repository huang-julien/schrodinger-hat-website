<script setup lang="ts">
import { type Ref, computed, ref, watch } from 'vue'
import { type RouteParamValue, useRoute } from 'vue-router'
import { useI18n } from 'vue-i18n'
import { useHead } from '@unhead/vue'
import * as messages from '../i18n/messages'
import { type EventMessageName } from '@/i18n/types'

const { t } = useI18n()
const route = useRoute()
const eventKey: Ref<RouteParamValue | null> = ref(null)

const filteredCtas = computed(() => {
  return messages.default.it.events[eventKey.value as EventMessageName].ctas.filter(cta => cta.value !== null)
})

const getCalendarLink = () => {
  let calendarString = 'http://www.google.com/calendar/event?action=TEMPLATE&dates='
  try {
    // @ts-expect-error Because of not typing messages
    const { title, location } = messages.it.events[eventKey.value as EventMessageName]
    // @ts-expect-error Because of not typing messages
    const [date, time] = messages.it.events[eventKey.value as EventMessageName].date.split(' ')
    const startUTCDate = new Date(`${date} ${time}`)
    const startDate = startUTCDate.toISOString().replace(/[:-]/g, '').replace('.000Z', 'Z')
    startUTCDate.setTime(startUTCDate.getTime() + 3600 * 2 * 1000)
    const endDate = startUTCDate.toISOString().replace(/[:-]/g, '').replace('.000Z', 'Z')
    calendarString += `${startDate}%2F${endDate}`
    calendarString += `&text=${title}&location=${location}&details=`
  }
  catch (err) {
    // TODO: Redirect to missing event page
    console.log(err)
  }
  return calendarString
}

watch(() => route.params.event, () => {
  eventKey.value = route.params.event as RouteParamValue
}, { immediate: true })

useHead({
  title: () => t(`events.${eventKey.value}.title`),
  meta: [{ name: t('head.events.meta.name'), content: t('head.events.meta.content') }],
})
</script>

<template>
  <!-- TODO: Add loader -->
  <span v-if="!eventKey">Add loader</span>
  <div v-else class="event">
    <div class="container">
      <div class="max-w-[700px] px-12 py-6 m-auto space-y-2">
        <h1 :data-test="`${eventKey}-title`" class="head-4">
          {{ t(`events.${eventKey}.title`) }}
        </h1>
        <h4>
          <a
            :data-test="`${eventKey}-date`"
            class="external-link-color"
            target="_blank"
            :href="getCalendarLink"
          >
            {{ t(`events.${eventKey}.date`) }}</a>
          |
          <a
            :data-test="`${eventKey}-location`"
            class="external-link-color"
            target="_blank"
            :href="t(`events.${eventKey}.location-link`)"
          >
            {{ t(`events.${eventKey}.location`) }}</a>
        </h4>
        <p :data-test="`${eventKey}-subtitle`">
          <i>{{ t(`events.${eventKey}.subtitle`) }}</i>
        </p>
        <div
          v-if="t(`events.${eventKey}.description`) !== ''"
          :data-test="`${eventKey}-description`"
          class="description"
          v-html="$t(`events.${eventKey}.description`)"
        />
        <div class="cta">
          <a
            v-for="{ id, value } in filteredCtas"
            :key="id"
            :data-test="`${eventKey}-${id}`"
            :href="value as string"
            class="btn btn-primary"
            target="_blank"
          > {{ t(`message.event.${id}`) }}</a>
        </div>
        <div
          v-if="t(`events.${eventKey}.sponsors`) !== ''"
          class="sponsors"
        >
          <br>
          <h3
            :data-test="`${eventKey}-sponsors-title`"
          >
            Sponsors
          </h3>
          <div
            class="sponsor-logos"
            :data-test="`${eventKey}-sponsors-logo`"
            v-html="t(`events.${eventKey}.sponsors`)"
          />
        </div>
        <div v-if="t(`events.${eventKey}.community-sponsors`) !== ''">
          <h3
            :data-test="`${eventKey}-community-sponsors-title`"
          >
            Community Sponsors
          </h3>
          <div
            :data-test="`${eventKey}-community-sponsors-logo`"
            class="sponsor-logos"
            v-html="t(`events.${eventKey}.community-sponsors`)"
          />
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped lang="scss">
  .cta {
    display: grid;
    margin-top: 20px;
    text-align: left;

    @include breakpoint(lg) {
      display: block;
    }

    .btn {
      margin: rem(4px) 0;

      @include breakpoint(lg) {
        margin: 0 rem(4px);

        &:nth-of-type(1) {
          margin-left: 0;
        }
      }
    }
  }

  .sponsor-logos {
    display: flex;
    flex: auto;
    flex-wrap: wrap;
    align-items: center;
    justify-content: center;
  }
</style>
