<script setup lang="ts">
import { useRoute } from 'vue-router'
import { computed } from 'vue'
import { User, Settings, CreditCard, Bell } from 'lucide-vue-next'

const route = useRoute()

const tabs = [
  { label: 'Profile',       path: '/user/profile',       icon: User },
  { label: 'Account',       path: '/user/account',       icon: Settings },
  { label: 'Billing',       path: '/user/billing',       icon: CreditCard },
  { label: 'Notifications', path: '/user/notifications', icon: Bell },
]

const active = computed(() => route.path)
</script>

<template>
  <div class="border-border border-b bg-card">
    <nav class="flex gap-0 px-6 overflow-x-auto">
      <RouterLink
        v-for="tab in tabs"
        :key="tab.path"
        :to="tab.path"
        class="flex items-center gap-2 border-b-2 px-4 py-3.5 text-sm font-medium whitespace-nowrap transition-colors"
        :class="active === tab.path
          ? 'border-primary text-primary'
          : 'border-transparent text-muted-foreground hover:text-foreground hover:border-border'"
      >
        <component :is="tab.icon" class="size-4" />
        {{ tab.label }}
      </RouterLink>
    </nav>
  </div>
</template>
