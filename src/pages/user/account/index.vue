<script setup lang="ts">
import { ref } from 'vue'
import { ShieldCheck, Smartphone, Monitor, Chrome, AlertTriangle } from 'lucide-vue-next'
import { Input } from '@/components/ui/input'
import { Button } from '@/components/ui/button'
import { Separator } from '@/components/ui/separator'
import Switch from '@/components/ui/switch/Switch.vue'
import UserSettingsNav from '@/components/UserSettingsNav.vue'

definePage({
  meta: { title: 'Account', description: 'Manage your account security and settings.' },
})

const email = ref('admin@example.com')
const newEmail = ref('')

const passwords = ref({ current: '', next: '', confirm: '' })

const twoFactor = ref(false)

const sessions = [
  { device: 'Chrome on macOS',   icon: Chrome,    location: 'San Francisco, CA', time: 'Active now',      current: true },
  { device: 'Safari on iPhone',  icon: Smartphone,location: 'San Francisco, CA', time: '2 hours ago',     current: false },
  { device: 'Firefox on Windows',icon: Monitor,   location: 'New York, NY',      time: '3 days ago',      current: false },
]
</script>

<template>
  <div class="flex flex-col">
    <UserSettingsNav />

    <div class="mx-auto w-full max-w-2xl space-y-8 p-6">

      <!-- Email -->
      <section class="bg-card border-border rounded-xl border p-6">
        <h2 class="mb-1 text-base font-semibold">Email Address</h2>
        <p class="text-muted-foreground mb-5 text-sm">
          Your current email is <span class="text-foreground font-medium">{{ email }}</span>.
        </p>
        <div class="space-y-3">
          <div class="space-y-1.5">
            <label class="text-sm font-medium">New email address</label>
            <Input v-model="newEmail" type="email" placeholder="new@example.com" />
          </div>
          <div class="space-y-1.5">
            <label class="text-sm font-medium">Confirm with password</label>
            <Input type="password" placeholder="Your current password" />
          </div>
          <Button size="sm">Update email</Button>
        </div>
      </section>

      <!-- Password -->
      <section class="bg-card border-border rounded-xl border p-6">
        <h2 class="mb-1 text-base font-semibold">Change Password</h2>
        <p class="text-muted-foreground mb-5 text-sm">
          Use a strong password that you don't use elsewhere.
        </p>
        <div class="space-y-3">
          <div class="space-y-1.5">
            <label class="text-sm font-medium">Current password</label>
            <Input v-model="passwords.current" type="password" placeholder="••••••••" />
          </div>
          <div class="space-y-1.5">
            <label class="text-sm font-medium">New password</label>
            <Input v-model="passwords.next" type="password" placeholder="••••••••" />
          </div>
          <div class="space-y-1.5">
            <label class="text-sm font-medium">Confirm new password</label>
            <Input v-model="passwords.confirm" type="password" placeholder="••••••••" />
          </div>

          <!-- Password strength bar -->
          <div class="space-y-1">
            <div class="flex gap-1">
              <div
                v-for="i in 4" :key="i"
                class="h-1 flex-1 rounded-full transition-colors"
                :class="passwords.next.length >= i * 3 ? 'bg-primary' : 'bg-muted'"
              />
            </div>
            <p class="text-muted-foreground text-xs">
              {{ passwords.next.length === 0 ? 'Enter a password' : passwords.next.length < 6 ? 'Too short' : passwords.next.length < 10 ? 'Fair' : passwords.next.length < 14 ? 'Good' : 'Strong' }}
            </p>
          </div>

          <Button size="sm">Update password</Button>
        </div>
      </section>

      <!-- Two-Factor Auth -->
      <section class="bg-card border-border rounded-xl border p-6">
        <div class="flex items-start justify-between gap-4">
          <div>
            <div class="mb-1 flex items-center gap-2">
              <ShieldCheck class="text-primary size-4" />
              <h2 class="text-base font-semibold">Two-Factor Authentication</h2>
            </div>
            <p class="text-muted-foreground text-sm">
              Add an extra layer of security to your account using an authenticator app.
            </p>
          </div>
          <Switch v-model="twoFactor" />
        </div>

        <template v-if="twoFactor">
          <Separator class="my-4" />
          <div class="bg-muted/50 border-border flex items-center gap-4 rounded-lg border p-4">
            <!-- QR code placeholder -->
            <div class="bg-white border-border flex size-20 shrink-0 items-center justify-center rounded-md border p-1">
              <div class="grid size-full grid-cols-5 gap-px">
                <div v-for="i in 25" :key="i"
                  class="rounded-sm"
                  :class="[1,2,3,4,5,6,10,11,15,16,20,21,22,23,24,25,7,13,19].includes(i) ? 'bg-black' : 'bg-white'" />
              </div>
            </div>
            <div>
              <p class="text-sm font-medium">Scan with your authenticator app</p>
              <p class="text-muted-foreground mt-1 text-xs">Or enter the code manually:</p>
              <code class="text-primary mt-1 block text-sm tracking-widest">JBSW Y3DP EBLW 64TM</code>
            </div>
          </div>
        </template>
      </section>

      <!-- Active Sessions -->
      <section class="bg-card border-border rounded-xl border p-6">
        <h2 class="mb-1 text-base font-semibold">Active Sessions</h2>
        <p class="text-muted-foreground mb-5 text-sm">
          Manage and revoke other browser or device sessions.
        </p>
        <div class="space-y-3">
          <div
            v-for="session in sessions"
            :key="session.device"
            class="flex items-center justify-between gap-4"
          >
            <div class="flex items-center gap-3">
              <div class="bg-muted flex size-9 items-center justify-center rounded-lg">
                <component :is="session.icon" class="text-muted-foreground size-4" />
              </div>
              <div>
                <div class="flex items-center gap-2">
                  <p class="text-sm font-medium">{{ session.device }}</p>
                  <span v-if="session.current"
                    class="bg-emerald-100 text-emerald-700 dark:bg-emerald-900/30 dark:text-emerald-400 rounded-full px-1.5 py-px text-[10px] font-medium">
                    Current
                  </span>
                </div>
                <p class="text-muted-foreground text-xs">{{ session.location }} · {{ session.time }}</p>
              </div>
            </div>
            <Button v-if="!session.current" variant="ghost" size="sm" class="text-destructive hover:text-destructive text-xs">
              Revoke
            </Button>
          </div>
        </div>
        <Separator class="my-4" />
        <Button variant="outline" size="sm">Revoke all other sessions</Button>
      </section>

      <!-- Danger Zone -->
      <section class="border-destructive/40 rounded-xl border p-6">
        <div class="mb-1 flex items-center gap-2">
          <AlertTriangle class="text-destructive size-4" />
          <h2 class="text-base font-semibold">Danger Zone</h2>
        </div>
        <p class="text-muted-foreground mb-5 text-sm">
          Once you delete your account, there is no going back. Please be certain.
        </p>
        <Button variant="destructive" size="sm">Delete my account</Button>
      </section>

    </div>
  </div>
</template>
