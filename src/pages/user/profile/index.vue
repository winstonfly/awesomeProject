<script setup lang="ts">
import { ref } from 'vue'
import { Camera, Trash2, Globe, MapPin, Building2, Link } from 'lucide-vue-next'
import { Avatar, AvatarFallback } from '@/components/ui/avatar'
import { Input } from '@/components/ui/input'
import { Button } from '@/components/ui/button'
import { Separator } from '@/components/ui/separator'
import UserSettingsNav from '@/components/UserSettingsNav.vue'

definePage({
  meta: { title: 'Profile', description: 'Manage your public profile information.' },
})

const form = ref({
  displayName: 'Admin User',
  username: 'adminuser',
  email: 'admin@example.com',
  bio: 'Full-stack developer and product designer. Building things for the web.',
  website: 'https://example.com',
  location: 'San Francisco, CA',
  company: 'Acme Inc.',
  twitter: '@adminuser',
  github: 'adminuser',
})

function save() { /* submit */ }
</script>

<template>
  <div class="flex flex-col">
    <UserSettingsNav />

    <div class="mx-auto w-full max-w-2xl space-y-8 p-6">

      <!-- Avatar Section -->
      <section class="bg-card border-border rounded-xl border p-6">
        <h2 class="mb-1 text-base font-semibold">Profile Picture</h2>
        <p class="text-muted-foreground mb-5 text-sm">
          This will be displayed on your profile and across the platform.
        </p>
        <div class="flex items-center gap-5">
          <div class="relative">
            <Avatar class="size-20">
              <AvatarFallback class="bg-primary text-primary-foreground text-xl font-bold">
                AU
              </AvatarFallback>
            </Avatar>
            <button
              class="bg-card border-border absolute -bottom-1 -right-1 flex size-7 items-center justify-center rounded-full border shadow-sm transition-colors hover:bg-muted"
            >
              <Camera class="size-3.5" />
            </button>
          </div>
          <div class="space-y-2">
            <div class="flex gap-2">
              <Button variant="outline" size="sm">Upload photo</Button>
              <Button variant="ghost" size="sm" class="text-destructive hover:text-destructive">
                <Trash2 class="mr-1.5 size-3.5" />
                Remove
              </Button>
            </div>
            <p class="text-muted-foreground text-xs">JPG, PNG or GIF · Max 5 MB</p>
          </div>
        </div>
      </section>

      <!-- Basic Info -->
      <section class="bg-card border-border rounded-xl border p-6">
        <h2 class="mb-1 text-base font-semibold">Basic Information</h2>
        <p class="text-muted-foreground mb-5 text-sm">Update your name and personal details.</p>

        <div class="space-y-4">
          <div class="grid gap-4 sm:grid-cols-2">
            <div class="space-y-1.5">
              <label class="text-sm font-medium">Display Name</label>
              <Input v-model="form.displayName" placeholder="Your name" />
            </div>
            <div class="space-y-1.5">
              <label class="text-sm font-medium">Username</label>
              <div class="relative">
                <span class="text-muted-foreground absolute left-3 top-1/2 -translate-y-1/2 text-sm">@</span>
                <Input v-model="form.username" class="pl-7" placeholder="username" />
              </div>
            </div>
          </div>

          <div class="space-y-1.5">
            <label class="text-sm font-medium">Bio</label>
            <textarea
              v-model="form.bio"
              rows="3"
              placeholder="Write a short bio..."
              class="border-input bg-background placeholder:text-muted-foreground focus-visible:ring-ring w-full rounded-md border px-3 py-2 text-sm shadow-sm focus-visible:outline-none focus-visible:ring-1 resize-none"
            />
            <p class="text-muted-foreground text-xs">{{ form.bio.length }} / 200 characters</p>
          </div>
        </div>
      </section>

      <!-- Contact & Links -->
      <section class="bg-card border-border rounded-xl border p-6">
        <h2 class="mb-1 text-base font-semibold">Contact & Links</h2>
        <p class="text-muted-foreground mb-5 text-sm">Where people can find you online.</p>

        <div class="space-y-4">
          <div class="space-y-1.5">
            <label class="text-sm font-medium flex items-center gap-1.5">
              <Globe class="size-3.5 text-muted-foreground" />
              Website
            </label>
            <Input v-model="form.website" placeholder="https://yoursite.com" type="url" />
          </div>

          <div class="grid gap-4 sm:grid-cols-2">
            <div class="space-y-1.5">
              <label class="text-sm font-medium flex items-center gap-1.5">
                <MapPin class="size-3.5 text-muted-foreground" />
                Location
              </label>
              <Input v-model="form.location" placeholder="City, Country" />
            </div>
            <div class="space-y-1.5">
              <label class="text-sm font-medium flex items-center gap-1.5">
                <Building2 class="size-3.5 text-muted-foreground" />
                Company
              </label>
              <Input v-model="form.company" placeholder="Company name" />
            </div>
          </div>

          <Separator />

          <div class="grid gap-4 sm:grid-cols-2">
            <div class="space-y-1.5">
              <label class="text-sm font-medium flex items-center gap-1.5">
                <Link class="size-3.5 text-muted-foreground" />
                Twitter / X
              </label>
              <Input v-model="form.twitter" placeholder="@handle" />
            </div>
            <div class="space-y-1.5">
              <label class="text-sm font-medium flex items-center gap-1.5">
                <Link class="size-3.5 text-muted-foreground" />
                GitHub
              </label>
              <Input v-model="form.github" placeholder="username" />
            </div>
          </div>
        </div>
      </section>

      <!-- Save -->
      <div class="flex justify-end gap-3">
        <Button variant="outline">Cancel</Button>
        <Button @click="save">Save changes</Button>
      </div>

    </div>
  </div>
</template>
