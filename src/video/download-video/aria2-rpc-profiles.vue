<template>
  <div class="aria2-rpc-profiles">
    <div class="profiles-header">
      <h2>预设</h2>
      <div class="profile-operations">
        <div
          v-if="profiles.length > 1"
          class="operation delete-profile"
          @click="deleteProfile()"
          title="删除预设"
        >
          <icon type="mdi" icon="trash-can-outline"></icon>
        </div>
        <div class="operation new-profile" @click="addProfile()" title="新增预设">
          <icon type="mdi" icon="plus"></icon>
        </div>
      </div>
    </div>
    <div class="profiles-list">
      <profile-item
        v-for="(profile, index) of profiles"
        :key="profile.name + index"
        @profile-update="profileUpdate()"
        @click.native="changeProfile(profile)"
        :profile="profile"
        :deletable="profiles.length > 1"
        :selected="profile.name === selectedProfile"
      ></profile-item>
    </div>
  </div>
</template>
<script lang="ts">
const defaultProfile: RpcOptionProfile = {
  name: '未命名',
  ...settings.aria2RpcOption
}
export default {
  components: {
    ProfileItem: () => import('./aria2-rpc-profile-item.vue'),
    Icon: () => import('../../style/icon.vue')
  },
  data() {
    this.migrateOldProfiles()
    const profiles = [...settings.aria2RpcOptionProfiles]
    if (profiles.length === 0) {
      profiles.push(defaultProfile)
      settings.aria2RpcOptionProfiles = profiles
    }
    return {
      profiles,
      selectedProfile:
        settings.aria2RpcOptionSelectedProfile || defaultProfile.name
    }
  },
  watch: {
    selectedProfile(newValue: string) {
      if (settings.aria2RpcOptionSelectedProfile !== newValue) {
        settings.aria2RpcOptionSelectedProfile = newValue
      }
    }
  },
  methods: {
    migrateOldProfiles() {
      const properties = Object.getOwnPropertyNames(
        settings.aria2RpcOption
      ).filter(it => !it.startsWith('_'))
      properties.push('name')
      let migrated = false
      for (const profile of settings.aria2RpcOptionProfiles) {
        properties
          .filter(p => !(p in profile))
          .forEach(p => {
            profile[p] = settings.aria2RpcOption[p]
            console.log(`migrated profile property '${p}'`)
            migrated = true
          })
      }
      if (migrated) {
        settings.aria2RpcOptionProfiles = settings.aria2RpcOptionProfiles
      }
    },
    profileUpdate() {
      settings.aria2RpcOptionProfiles = this.profiles
      this.selectedProfile = settings.aria2RpcOptionSelectedProfile
    },
    changeProfile(profile: RpcOptionProfile) {
      this.selectedProfile = profile.name
      this.$emit('profile-change', profile)
    },
    addProfile() {
      const profile: RpcOptionProfile = {
        ...this.profiles.find(
          (p: RpcOptionProfile) => p.name === this.selectedProfile
        )
      }
      profile.name = profile.name.replace(/[\d]+$/, '')
      if (
        this.profiles.some((p: RpcOptionProfile) => p.name === profile.name)
      ) {
        let suffix = 1
        while (
          this.profiles.some(
            (p: RpcOptionProfile) => p.name === profile.name + suffix.toString()
          )
        ) {
          suffix++
        }
        profile.name = profile.name + suffix.toString()
      }
      this.profiles.push(profile)
      settings.aria2RpcOptionProfiles = this.profiles
      this.changeProfile(profile)
    },
    deleteProfile() {
      const selectedIndex = (this.profiles as RpcOptionProfile[]).findIndex(
        p => p.name === this.selectedProfile
      )
      const otherIndex = selectedIndex === 0 ? 0 : selectedIndex - 1
      const otherProfile = this.profiles[otherIndex]
      this.profiles.splice(selectedIndex, 1)
      settings.aria2RpcOptionProfiles = this.profiles
      this.changeProfile(otherProfile)
    }
  }
}
</script>
<style lang="scss">
.aria2-rpc-profiles {
  .profiles-header {
    display: flex;
    justify-content: space-between;
    align-items: center;
    margin-bottom: 8px;
    .profile-operations {
      display: flex;
      align-items: center;
      .operation {
        padding: 4px;
        background-color: #8882;
        border-radius: 50%;
        display: flex;
        align-items: center;
        justify-content: center;
        cursor: pointer;
        &:not(:last-child) {
          margin-right: 8px;
        }
        &:hover {
          background-color: #8884;
        }
        .mdi {
          margin: 0;
        }
      }
    }
  }
  .profiles-list {
    display: flex;
    overflow: auto;
    scrollbar-width: none !important;
    &::-webkit-scrollbar {
      height: 0px !important;
    }
  }
}
</style>