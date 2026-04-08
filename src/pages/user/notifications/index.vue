<script setup lang="ts">
import { ref } from 'vue'
import { Button } from '@/components/ui/button'
import { Separator } from '@/components/ui/separator'
import Switch from '@/components/ui/switch/Switch.vue'
import UserSettingsNav from '@/components/UserSettingsNav.vue'
import {
  MessageSquare, AtSign, Heart, UserPlus,
  ShieldAlert, Megaphone, Mail, Smartphone, Bell,
} from 'lucide-vue-next'

definePage({
  meta: { title: 'Notifications', description: 'Choose what you want to be notified about.' },
})

interface NotifSetting {
  label: string
  description: string
  icon: typeof Bell
  email: boolean
  push: boolean
  inApp: boolean
}

const settings = ref<NotifSetting[]>([
  {
    label: 'Comments',
    description: 'When someone comments on your posts or projects.',
    icon: MessageSquare,
    email: true, push: true, inApp: true,
  },
  {
    label: 'Mentions',
    description: 'When someone mentions you with @username.',
    icon: AtSign,
    email: true, push: true, inApp: true,
  },
  {
    label: 'Likes & Reactions',
    description: 'When someone likes or reacts to your content.',
    icon: Heart,
    email: false, push: true, inApp: true,
  },
  {
    label: 'New Followers',
    description: 'When someone starts following you.',
    icon: UserPlus,
    email: false, push: false, inApp: true,
  },
  {
    label: 'Security Alerts',
    description: 'Important alerts about your account security.',
    icon: ShieldAlert,
    email: true, push: true, inApp: true,
  },
  {
    label: 'Product Updates',
    description: 'New features, releases, and product announcements.',
    icon: Megaphone,
    email: true, push: false, inApp: false,
  },
])

// Channel-level master toggles
const channels = ref({ email: true, push: true, inApp: true })

function toggleAll(channel: 'email' | 'push' | 'inApp') {
  const next = !channels.value[channel]
  channels.value[channel] = next
  settings.value.forEach(s => { s[channel] = next })
}

function save() { /* submit */ }
</script>

<template>
  <div class="flex flex-col">
    <UserSettingsNav />

    <div class="mx-auto w-full max-w-2xl space-y-8 p-6">

      <!-- Channel toggles -->
      <section class="bg-card border-border rounded-xl border p-6">
        <h2 class="mb-1 text-base font-semibold">Notification Channels</h2>
        <p class="text-muted-foreground mb-5 text-sm">
          Master switches for each delivery channel.
        </p>
        <div class="grid gap-4 sm:grid-cols-3">
          <div
            v-for="(ch, key) in [
              { key: 'email', label: 'Email',        icon: Mail,       value: channels.email },
              { key: 'push',  label: 'Push',         icon: Smartphone, value: channels.push  },
              { key: 'inApp', label: 'In-App',       icon: Bell,       value: channels.inApp },
            ]"
            :key="ch.key"
            class="border-border flex items-center gap-3 rounded-xl border p-4"
          >
            <div class="bg-muted flex size-9 items-center justify-center rounded-lg">
              <component :is="ch.icon" class="text-muted-foreground size-4" />
            </div>
            <div class="flex-1">
              <p class="text-sm font-medium">{{ ch.label }}</p>
              <p class="text-muted-foreground text-xs">{{ ch.value ? 'Enabled' : 'Disabled' }}</p>
            </div>
            <Switch
              :model-value="ch.value"
              @update:model-value="toggleAll(ch.key as 'email' | 'push' | 'inApp')"
            />
          </div>
        </div>
      </section>

      <!-- Per-event settings -->
      <section class="bg-card border-border rounded-xl border">
        <div class="border-border border-b px-6 py-4">
          <h2 class="text-base font-semibold">Notification Preferences</h2>
          <p class="text-muted-foreground text-sm">Choose how you're notified for each event.</p>
        </div>

        <!-- Header row -->
        <div class="border-border grid grid-cols-[1fr_auto_auto_auto] items-center gap-4 border-b bg-muted/30 px-6 py-2.5">
          <span class="text-muted-foreground text-xs font-medium uppercase tracking-wider">Event</span>
          <span class="text-muted-foreground w-12 text-center text-xs font-medium uppercase tracking-wider">Email</span>
          <span class="text-muted-foreground w-12 text-center text-xs font-medium uppercase tracking-wider">Push</span>
          <span class="text-muted-foreground w-12 text-center text-xs font-medium uppercase tracking-wider">In-App</span>
        </div>

        <div>
          <div
            v-for="(item, i) in settings"
            :key="item.label"
            class="grid grid-cols-[1fr_auto_auto_auto] items-center gap-4 px-6 py-4 transition-colors hover:bg-muted/20"
            :class="i < settings.length - 1 ? 'border-b border-border' : ''"
          >
            <div class="flex items-center gap-3 min-w-0">
              <div class="bg-muted flex size-8 shrink-0 items-center justify-center rounded-lg">
                <component :is="item.icon" class="text-muted-foreground size-4" />
              </div>
              <div class="min-w-0">
                <p class="text-sm font-medium">{{ item.label }}</p>
                <p class="text-muted-foreground truncate text-xs">{{ item.description }}</p>
              </div>
            </div>
            <div class="flex w-12 justify-center">
              <Switch v-model="item.email" />
            </div>
            <div class="flex w-12 justify-center">
              <Switch v-model="item.push" />
            </div>
            <div class="flex w-12 justify-center">
              <Switch v-model="item.inApp" />
            </div>
          </div>
        </div>
      </section>

      <!-- Digest frequency -->
      <section class="bg-card border-border rounded-xl border p-6">
        <h2 class="mb-1 text-base font-semibold">Email Digest</h2>
        <p class="text-muted-foreground mb-4 text-sm">
          Receive a digest of your notifications instead of individual emails.
        </p>
        <div class="grid grid-cols-3 gap-2">
          <button
            v-for="opt in ['Real-time', 'Daily', 'Weekly']"
            :key="opt"
            class="border-border rounded-lg border py-2.5 text-sm font-medium transition-colors"
            :class="opt === 'Daily'
              ? 'border-primary bg-primary/8 text-primary'
              : 'text-muted-foreground hover:text-foreground hover:border-foreground/20'"
          >
            {{ opt }}
          </button>
        </div>
      </section>

      <!-- Save -->
      <div class="flex justify-end gap-3">
        <Button variant="outline">Cancel</Button>
        <Button @click="save">Save preferences</Button>
      </div>

    </div>
  </div>
</template>
