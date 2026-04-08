<script setup lang="ts">
import { ref } from 'vue'
import { CreditCard, Download, Zap, Check, ArrowUpRight } from 'lucide-vue-next'
import { Button } from '@/components/ui/button'
import { Separator } from '@/components/ui/separator'
import UserSettingsNav from '@/components/UserSettingsNav.vue'

definePage({
  meta: { title: 'Billing', description: 'Manage your subscription and payment methods.' },
})

const currentPlan = ref<'free' | 'pro' | 'enterprise'>('pro')

const plans = [
  {
    id: 'free',
    name: 'Free',
    price: '$0',
    period: '/ month',
    description: 'For individuals getting started.',
    features: ['Up to 5 projects', '1 GB storage', 'Basic analytics', 'Email support'],
  },
  {
    id: 'pro',
    name: 'Pro',
    price: '$12',
    period: '/ month',
    description: 'For professionals and small teams.',
    features: ['Unlimited projects', '50 GB storage', 'Advanced analytics', 'Priority support', 'API access', 'Custom domains'],
    badge: 'Current plan',
  },
  {
    id: 'enterprise',
    name: 'Enterprise',
    price: '$49',
    period: '/ month',
    description: 'For large teams and organizations.',
    features: ['Everything in Pro', '500 GB storage', 'Dedicated support', 'SSO / SAML', 'Audit logs', 'SLA guarantee'],
  },
]

const invoices = [
  { id: 'INV-2024-012', date: 'Dec 1, 2024', amount: '$12.00', status: 'Paid' },
  { id: 'INV-2024-011', date: 'Nov 1, 2024', amount: '$12.00', status: 'Paid' },
  { id: 'INV-2024-010', date: 'Oct 1, 2024', amount: '$12.00', status: 'Paid' },
  { id: 'INV-2024-009', date: 'Sep 1, 2024', amount: '$12.00', status: 'Paid' },
  { id: 'INV-2024-008', date: 'Aug 1, 2024', amount: '$12.00', status: 'Paid' },
]
</script>

<template>
  <div class="flex flex-col">
    <UserSettingsNav />

    <div class="mx-auto w-full max-w-3xl space-y-8 p-6">

      <!-- Current Plan Summary -->
      <section class="bg-card border-border rounded-xl border p-6">
        <div class="flex items-start justify-between gap-4">
          <div>
            <div class="mb-1 flex items-center gap-2">
              <Zap class="text-primary size-4" />
              <h2 class="text-base font-semibold">Current Plan</h2>
            </div>
            <p class="text-muted-foreground text-sm">
              You're on the <span class="text-foreground font-semibold">Pro</span> plan.
              Your next billing date is <span class="text-foreground font-semibold">January 1, 2025</span>.
            </p>
          </div>
          <span class="bg-primary/10 text-primary rounded-full px-3 py-1 text-sm font-semibold">Pro</span>
        </div>

        <Separator class="my-4" />

        <!-- Usage bar -->
        <div class="space-y-3">
          <div class="flex justify-between text-sm">
            <span class="text-muted-foreground">Storage used</span>
            <span class="font-medium">18.4 GB <span class="text-muted-foreground">/ 50 GB</span></span>
          </div>
          <div class="bg-muted h-2 overflow-hidden rounded-full">
            <div class="bg-primary h-full rounded-full" style="width: 37%" />
          </div>
          <div class="flex justify-between text-sm">
            <span class="text-muted-foreground">Projects</span>
            <span class="font-medium">12 <span class="text-muted-foreground">/ Unlimited</span></span>
          </div>
        </div>
      </section>

      <!-- Plans -->
      <section>
        <h2 class="mb-4 text-base font-semibold">Available Plans</h2>
        <div class="grid gap-4 md:grid-cols-3">
          <div
            v-for="plan in plans"
            :key="plan.id"
            class="bg-card border-border relative flex flex-col rounded-xl border p-5 transition-shadow"
            :class="plan.id === 'pro' ? 'ring-primary ring-2' : ''"
          >
            <div v-if="plan.badge"
              class="bg-primary text-primary-foreground absolute -top-3 left-1/2 -translate-x-1/2 rounded-full px-3 py-0.5 text-xs font-semibold whitespace-nowrap">
              {{ plan.badge }}
            </div>

            <div class="mb-3">
              <p class="text-sm font-semibold">{{ plan.name }}</p>
              <div class="mt-1 flex items-baseline gap-1">
                <span class="text-2xl font-bold">{{ plan.price }}</span>
                <span class="text-muted-foreground text-sm">{{ plan.period }}</span>
              </div>
              <p class="text-muted-foreground mt-1 text-xs">{{ plan.description }}</p>
            </div>

            <ul class="mb-5 flex-1 space-y-2">
              <li v-for="f in plan.features" :key="f" class="flex items-center gap-2 text-sm">
                <Check class="text-primary size-3.5 shrink-0" />
                {{ f }}
              </li>
            </ul>

            <Button
              :variant="plan.id === currentPlan ? 'outline' : plan.id === 'enterprise' ? 'default' : 'default'"
              size="sm"
              :disabled="plan.id === currentPlan"
              class="w-full"
            >
              {{ plan.id === currentPlan ? 'Current plan' : plan.id === 'free' ? 'Downgrade' : 'Upgrade' }}
            </Button>
          </div>
        </div>
      </section>

      <!-- Payment Method -->
      <section class="bg-card border-border rounded-xl border p-6">
        <div class="mb-5 flex items-center justify-between">
          <div>
            <h2 class="text-base font-semibold">Payment Method</h2>
            <p class="text-muted-foreground text-sm">Update your billing details and card information.</p>
          </div>
          <Button variant="outline" size="sm">Update card</Button>
        </div>

        <div class="border-border bg-muted/30 flex items-center gap-4 rounded-xl border p-4">
          <div class="bg-background border-border flex size-12 items-center justify-center rounded-lg border">
            <CreditCard class="text-muted-foreground size-5" />
          </div>
          <div>
            <p class="text-sm font-medium">Visa ending in 4242</p>
            <p class="text-muted-foreground text-xs">Expires 08 / 2026 · Billing to admin@example.com</p>
          </div>
          <span class="ml-auto rounded-full bg-emerald-100 px-2 py-0.5 text-xs font-medium text-emerald-700 dark:bg-emerald-900/30 dark:text-emerald-400">
            Default
          </span>
        </div>
      </section>

      <!-- Invoice History -->
      <section class="bg-card border-border rounded-xl border">
        <div class="border-border flex items-center justify-between border-b px-6 py-4">
          <div>
            <h2 class="text-base font-semibold">Billing History</h2>
            <p class="text-muted-foreground text-sm">Download past invoices.</p>
          </div>
          <Button variant="outline" size="sm" class="gap-1.5">
            <Download class="size-3.5" />
            Export all
          </Button>
        </div>

        <table class="w-full text-sm">
          <thead>
            <tr class="border-border border-b">
              <th class="text-muted-foreground px-6 py-3 text-left text-xs font-medium uppercase tracking-wider">Invoice</th>
              <th class="text-muted-foreground px-4 py-3 text-left text-xs font-medium uppercase tracking-wider">Date</th>
              <th class="text-muted-foreground px-4 py-3 text-left text-xs font-medium uppercase tracking-wider">Amount</th>
              <th class="text-muted-foreground px-4 py-3 text-left text-xs font-medium uppercase tracking-wider">Status</th>
              <th class="px-6 py-3" />
            </tr>
          </thead>
          <tbody>
            <tr
              v-for="inv in invoices"
              :key="inv.id"
              class="border-border hover:bg-muted/30 border-b transition-colors last:border-0"
            >
              <td class="px-6 py-3 font-medium">{{ inv.id }}</td>
              <td class="text-muted-foreground px-4 py-3">{{ inv.date }}</td>
              <td class="px-4 py-3 font-medium">{{ inv.amount }}</td>
              <td class="px-4 py-3">
                <span class="rounded-full bg-emerald-100 px-2.5 py-0.5 text-xs font-medium text-emerald-700 dark:bg-emerald-900/30 dark:text-emerald-400">
                  {{ inv.status }}
                </span>
              </td>
              <td class="px-6 py-3 text-right">
                <button class="text-muted-foreground hover:text-foreground flex items-center gap-1 text-xs transition-colors ml-auto">
                  <Download class="size-3" />
                  PDF
                </button>
              </td>
            </tr>
          </tbody>
        </table>
      </section>

      <!-- Cancel Plan -->
      <div class="text-center">
        <button class="text-muted-foreground hover:text-destructive text-sm underline-offset-4 transition-colors hover:underline">
          Cancel subscription
        </button>
      </div>

    </div>
  </div>
</template>
