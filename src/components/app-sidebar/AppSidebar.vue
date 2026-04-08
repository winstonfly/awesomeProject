<script setup lang="ts">
import type { SidebarProps } from "@/components/ui/sidebar"

import {
  AudioWaveform,
  BookOpen,
  Bot,
  Command,
  Frame,
  GalleryVerticalEnd,
  LayoutDashboard,
  Map,
  PieChart,
  Settings2,
  ShoppingCart,
  SquareTerminal,
  Server,
  KeyRound,
  Layers,
  Activity,
  Users,
  GitFork,
} from "lucide-vue-next"
import NavMain from "@/components/app-sidebar/NavMain.vue"
import NavProjects from "@/components/app-sidebar/NavProjects.vue"
import NavUser from "@/components/app-sidebar/NavUser.vue"
import TeamSwitcher from "@/components/app-sidebar/TeamSwitcher.vue"

import {
  Sidebar,
  SidebarContent,
  SidebarFooter,
  SidebarHeader,
  SidebarRail,
} from "@/components/ui/sidebar"

const props = withDefaults(defineProps<SidebarProps>(), {
  collapsible: "icon",
})

// This is sample data.
const data = {
  user: {
    name: "shadcn",
    email: "m@example.com",
    avatar: "https://github.com/shadcn.png",
  },
  teams: [
    {
      name: "北京测试空间",
      logo: GalleryVerticalEnd,
      plan: "Enterprise",
    },
    {
      name: "Acme Corp.",
      logo: AudioWaveform,
      plan: "Startup",
    },
    {
      name: "Evil Corp.",
      logo: Command,
      plan: "Free",
    },
  ],
  navMain: [
    {
      title: "Steps",
      url: "/manage/stepper",
      items: [],
    },
    {
      title: "Dashboard",
      url: "/dashboard",
      icon: LayoutDashboard,
      items: [],
    },
    {
      title: "Management",
      url: "#",
      icon: Server,
      isActive: true,
      items: [
        {
          title: "Users",
          url: "/manage/users",
        },
        {
          title: "topology",
          url: "/manage/topology",
        },
        {
          title: "Nodes",
          url: "/manage/nodes",
        },
        {
          title: "Workspaces",
          url: "/manage/workspaces",
        },
        {
          title: "Peering",
          url: "/manage/peers",
        },
      ],
    },
    {
      title: "Settings",
      url: "#",
      icon: Settings2,
      items: [
        {
          title: "General",
          url: "#",
        },
        {
          title: "Team",
          url: "#",
        },
        {
          title: "Billing",
          url: "#",
        },
        {
          title: "Limits",
          url: "#",
        },
      ],
    },
  ],
}
</script>

<template>
  <Sidebar v-bind="props">
    <SidebarHeader>
      <TeamSwitcher :teams="data.teams" />
    </SidebarHeader>
    <SidebarContent>
      <NavMain :items="data.navMain" />
<!--      <NavMain label="网络管理" :items="data.navNetwork" />-->
<!--      <NavProjects :projects="data.projects" />-->
    </SidebarContent>
    <SidebarFooter>
      <NavUser :user="data.user" />
    </SidebarFooter>
    <SidebarRail />
  </Sidebar>
</template>
