<template>
  <v-container fluid class="px-6 pb-6">
    <v-row class="mb-4" align="center">
      <v-col cols="12" md="6">
        <h1 class="text-h4 font-weight-bold">Team Composition</h1>
        <p class="text-subtitle-1 text-medium-emphasis">Map engineers to their respective teams via drag and drop.</p>
      </v-col>
      <v-col cols="12" md="6" class="d-flex align-center justify-end">
        <!-- Add Member button -->
        <v-btn
          color="primary"
          prepend-icon="mdi-account-plus"
          rounded="pill"
          variant="flat"
          height="40"
          class="font-weight-bold flex-shrink-0 mr-3"
          @click="addDialog = true"
        >
          Add Member
        </v-btn>

        <!-- CSV import pill -->
        <div class="d-flex align-center" style="height: 40px; border: 1px solid rgba(255,255,255,0.12); border-radius: 20px; overflow: hidden; background: rgba(255,255,255,0.04);">
          <!-- Role Dropdown Menu -->
          <v-menu offset-y>
            <template v-slot:activator="{ props }">
              <v-btn
                variant="text"
                v-bind="props"
                append-icon="mdi-chevron-down"
                height="40"
                class="text-none font-weight-medium px-4 rounded-0"
                style="border-radius: 20px 0 0 20px !important; min-width: 140px;"
              >
                {{ csvRole === 'member' ? 'Intern / Member' : 'Mentor' }}
              </v-btn>
            </template>
            <v-list density="compact" class="py-1" elevation="8" rounded="lg">
              <v-list-item
                v-for="item in roleOptions"
                :key="item.value"
                @click="csvRole = item.value"
                :active="csvRole === item.value"
                color="primary"
              >
                <v-list-item-title class="text-body-2">{{ item.title }}</v-list-item-title>
              </v-list-item>
            </v-list>
          </v-menu>
          <div style="width: 1px; height: 24px; background: rgba(255,255,255,0.15); flex-shrink: 0;"></div>
          <input
            ref="csvInputRef"
            type="file"
            accept=".csv"
            style="display: none;"
            @change="handleFileUpload"
          />
          <v-btn
            prepend-icon="mdi-file-upload-outline"
            variant="text"
            color="primary"
            height="40"
            class="font-weight-medium px-3 rounded-0"
            style="border-radius: 0 20px 20px 0 !important;"
            @click="csvInputRef.click()"
          >
            Upload CSV
          </v-btn>
        </div>
      </v-col>
    </v-row>

    <!-- Add Member Dialog -->
    <v-dialog v-model="addDialog" max-width="440" persistent>
      <v-card rounded="xl" class="pa-2">
        <v-card-title class="text-h6 font-weight-bold d-flex align-center pt-4 px-5">
          <v-icon color="primary" class="mr-2">mdi-account-plus</v-icon>
          Add Member
        </v-card-title>
        <v-card-text class="px-5 pb-2">
          <v-form ref="addForm" @submit.prevent="addMemberManually">
            <v-text-field
              v-model="newMember.name"
              label="Full Name"
              prepend-inner-icon="mdi-account"
              variant="outlined"
              density="comfortable"
              rounded="lg"
              class="mb-3"
              :rules="[v => !!v || 'Name is required']"
              required
            ></v-text-field>
            <v-text-field
              v-model="newMember.email"
              label="Email"
              prepend-inner-icon="mdi-email"
              variant="outlined"
              density="comfortable"
              rounded="lg"
              class="mb-3"
              type="email"
              :rules="[v => !!v || 'Email is required', v => /.+@.+\..+/.test(v) || 'Enter a valid email']"
              required
            ></v-text-field>
            <v-text-field
              v-model="newMember.location"
              label="Location (City)"
              prepend-inner-icon="mdi-map-marker"
              variant="outlined"
              density="comfortable"
              rounded="lg"
              class="mb-3"
              :rules="[v => !!v || 'Location is required']"
              required
            ></v-text-field>
            <v-select
              v-model="newMember.role"
              :items="roleOptions"
              label="Role"
              prepend-inner-icon="mdi-shield-account"
              variant="outlined"
              density="comfortable"
              rounded="lg"
              class="mb-1"
            ></v-select>
          </v-form>
        </v-card-text>
        <v-card-actions class="px-5 pb-4">
          <v-spacer></v-spacer>
          <v-btn variant="text" rounded="pill" @click="closeAddDialog">Cancel</v-btn>
          <v-btn color="primary" variant="flat" rounded="pill" class="font-weight-bold" :loading="addLoading" @click="addMemberManually">Add Member</v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>

    <!-- Unassigned Pool -->
    <v-row class="pb-4" align="start">
      <!-- Mentors Section -->
      <v-col cols="12" md="6">
        <v-expansion-panels v-model="unassignedMentorsPanel" multiple>
          <v-expansion-panel class="glass-panel" elevation="0" rounded="xl" value="mentors">
            <v-expansion-panel-title class="py-3 font-weight-bold text-white">
              <div class="d-flex align-center">
                <v-icon start color="secondary">mdi-shield-star</v-icon>
                Unassigned Mentors
                <v-chip size="x-small" color="secondary" variant="tonal" class="ml-2 font-weight-bold">{{ getUnassignedByRole('mentor').length }}</v-chip>
              </div>
            </v-expansion-panel-title>
            <v-expansion-panel-text class="pt-3 list-container">
              <draggable
                :list="getUnassignedByRole('mentor')"
                item-key="id"
                group="engineers"
                @change="onChange($event, null, 'mentor')"
                class="d-flex flex-wrap align-center"
                style="gap: 8px; min-height: 60px; padding-bottom: 4px;"
              >
                <template #item="{ element }">
                  <v-card class="cursor-move flex-shrink-0 engineer-pill" rounded="xl" elevation="0" width="200" style="border: 1px solid #9C27B0;">
                    <div class="pill-glow" style="background: linear-gradient(90deg, transparent, #9C27B033)"></div>
                    <v-card-item class="py-1 px-2 position-relative z-1">
                      <div class="d-flex justify-space-between align-center">
                        <div class="d-flex align-center">
                          <v-avatar size="28" color="secondary" class="mr-2">
                            <v-icon size="14" color="white">mdi-shield-star</v-icon>
                          </v-avatar>
                          <div>
                            <div class="text-caption font-weight-bold lh-sm">{{ element.name }}</div>
                            <div v-if="element.location" class="text-grey-lighten-1 d-flex align-center mt-1" style="font-size: 0.65rem; line-height: 1;">
                              <v-icon size="8" class="mr-1">mdi-map-marker</v-icon> {{ element.location }}
                            </div>
                          </div>
                        </div>
                        <v-btn icon="mdi-close-circle" variant="text" size="x-small" color="error" class="ml-1" @click.stop="deleteMember(element.id)"></v-btn>
                      </div>
                    </v-card-item>
                  </v-card>
                </template>
              </draggable>
              <div v-if="!getUnassignedByRole('mentor').length" class="text-caption text-grey text-center py-3">No unassigned mentors</div>
            </v-expansion-panel-text>
          </v-expansion-panel>
        </v-expansion-panels>
      </v-col>

      <!-- Interns / Members Section -->
      <v-col cols="12" md="6">
        <v-expansion-panels v-model="unassignedInternsPanel" multiple>
          <v-expansion-panel class="glass-panel" elevation="0" rounded="xl" value="members">
            <v-expansion-panel-title class="py-3 font-weight-bold text-white">
              <div class="d-flex align-center">
                <v-icon start color="primary">mdi-account-group</v-icon>
                Unassigned Interns
                <v-chip size="x-small" color="primary" variant="tonal" class="ml-2 font-weight-bold">{{ getUnassignedByRole('member').length }}</v-chip>
              </div>
            </v-expansion-panel-title>
            <v-expansion-panel-text class="pt-3 list-container">
              <draggable
                :list="getUnassignedByRole('member')"
                item-key="id"
                group="engineers"
                @change="onChange($event, null, 'member')"
                class="d-flex flex-wrap align-center"
                style="gap: 8px; min-height: 60px; padding-bottom: 4px;"
              >
                <template #item="{ element }">
                  <v-card class="cursor-move flex-shrink-0 engineer-pill" rounded="xl" elevation="0" width="200">
                    <div class="pill-glow"></div>
                    <v-card-item class="py-1 px-2 position-relative z-1">
                      <div class="d-flex justify-space-between align-center">
                        <div class="d-flex align-center">
                          <v-avatar size="28" color="primary" class="mr-2">
                            <span class="text-caption font-weight-bold">{{ element.name.charAt(0) }}</span>
                          </v-avatar>
                          <div>
                            <div class="text-caption font-weight-bold lh-sm">{{ element.name }}</div>
                            <div v-if="element.location" class="text-grey-lighten-1 d-flex align-center mt-1" style="font-size: 0.65rem; line-height: 1;">
                              <v-icon size="8" class="mr-1">mdi-map-marker</v-icon> {{ element.location }}
                            </div>
                          </div>
                        </div>
                        <v-btn icon="mdi-close-circle" variant="text" size="x-small" color="error" class="ml-1" @click.stop="deleteMember(element.id)"></v-btn>
                      </div>
                    </v-card-item>
                  </v-card>
                </template>
              </draggable>
              <div v-if="!getUnassignedByRole('member').length" class="text-caption text-grey text-center py-3">No unassigned interns</div>
            </v-expansion-panel-text>
          </v-expansion-panel>
        </v-expansion-panels>
      </v-col>
    </v-row>

    <!-- Teams -->
    <v-row class="pb-4 flex-wrap list-container">
      <v-col v-for="teamId in 6" :key="teamId" cols="12" sm="6" md="4" lg="2">
        <v-card class="glass-column d-flex flex-column drop-zone h-100" :style="`--team-color: ${getTeamColor(teamId)}`" elevation="0" rounded="lg">
          <v-card-title class="py-2 px-3 d-flex justify-space-between align-center" style="min-height: 40px;">
            <div class="d-flex align-center">
              <div class="status-dot mr-2" :style="`background-color: ${getTeamColor(teamId)}; box-shadow: 0 0 6px ${getTeamColor(teamId)}; width: 6px; height: 6px;`"></div>
              <span class="text-body-2 font-weight-bold text-white letter-spacing-1">Team {{ teamId }}</span>
            </div>
            <div class="d-flex align-center gap-1">
              <v-chip size="x-small" variant="tonal" :color="getTeamColor(teamId)" class="font-weight-bold px-2">{{ getEngineersByTeam(teamId).length + getEngineersByTeamAndRole(teamId, 'mentor').length }}</v-chip>
              <v-btn
                icon="mdi-restore"
                variant="text"
                size="x-small"
                color="grey-lighten-1"
                class="ml-1 opacity-60 hover-opacity-100"
                style="width: 22px; height: 22px;"
                @click.stop="resetTeam(teamId)"
              ></v-btn>
            </div>
          </v-card-title>
          <v-card-text class="flex-grow-1 pa-2 list-container" style="overflow: visible;">
            <!-- Mentors Zone -->
            <div class="text-overline font-weight-bold text-secondary mb-1 d-flex align-center lh-sm" style="line-height: 1;">
              <v-icon size="x-small" class="mr-1">mdi-shield-star</v-icon> Mentors
            </div>
            <draggable
              :list="getEngineersByTeamAndRole(teamId, 'mentor')"
              item-key="id"
              group="engineers"
              @change="onChange($event, teamId, 'mentor')"
              class="mb-2"
              style="min-height: 40px; border: 1px dashed rgba(255,255,255,0.2); border-radius: 6px; padding: 2px;"
            >
              <template #item="{ element }">
                <v-card class="mb-1 cursor-move engineer-pill" rounded="pill" elevation="0" style="border: 1px solid #9C27B0">
                  <div class="pill-glow" :style="`background: linear-gradient(90deg, transparent, #9C27B033)`"></div>
                  <v-card-item class="pa-1 px-2 position-relative z-1">
                    <div class="d-flex align-center justify-space-between">
                      <div class="d-flex align-center">
                        <v-avatar size="20" color="secondary" class="mr-2">
                          <v-icon size="12" color="white">mdi-shield-star</v-icon>
                        </v-avatar>
                        <div>
                          <div class="text-caption font-weight-bold lh-sm" style="font-size: 0.7rem !important;">{{ element.name }}</div>
                        </div>
                      </div>
                      <v-btn icon="mdi-undo-variant" variant="text" size="x-small" color="grey-lighten-1" class="ml-1" @click.stop="revertMember(element.id)" style="width: 20px; height: 20px;"></v-btn>
                    </div>
                  </v-card-item>
                </v-card>
              </template>
            </draggable>

            <!-- Members Zone -->
            <div class="text-overline font-weight-bold text-grey-lighten-1 mb-1 d-flex align-center lh-sm" style="line-height: 1;">
              <v-icon size="x-small" class="mr-1">mdi-account-group</v-icon> Members
            </div>
            <draggable
              :list="getEngineersByTeamAndRole(teamId, 'member')"
              item-key="id"
              group="engineers"
              @change="onChange($event, teamId, 'member')"
              class="h-100 min-h-100px"
              style="min-height: 60px;"
            >
              <template #item="{ element }">
                <v-card class="mb-2 cursor-move engineer-pill" rounded="xl" elevation="0">
                  <div class="pill-glow" :style="`background: linear-gradient(90deg, transparent, ${getTeamColor(teamId)}33)`"></div>
                  <v-card-item class="py-1 px-2 position-relative z-1">
                    <div class="d-flex justify-space-between align-center">
                      <div class="d-flex align-center">
                        <v-avatar size="28" :color="getTeamColor(teamId)" class="mr-2">
                          <span class="text-caption font-weight-bold text-white">{{ element.name.charAt(0) }}</span>
                        </v-avatar>
                        <div>
                          <div class="text-caption font-weight-bold lh-sm">{{ element.name }}</div>
                          <div v-if="element.location" class="text-grey-lighten-1 d-flex align-center mt-1" style="font-size: 0.65rem; line-height: 1;">
                            <v-icon size="8" class="mr-1">mdi-map-marker</v-icon> {{ element.location }}
                          </div>
                        </div>
                      </div>
                      <v-btn icon="mdi-undo-variant" variant="text" size="x-small" color="grey-lighten-1" class="ml-1" @click.stop="revertMember(element.id)" style="width: 24px; height: 24px;"></v-btn>
                    </div>
                  </v-card-item>
                </v-card>
              </template>
            </draggable>
          </v-card-text>
        </v-card>
      </v-col>
    </v-row>
  </v-container>
</template>

<script setup>
import { ref, onMounted, watch } from 'vue'
import draggable from 'vuedraggable'
import Papa from 'papaparse'
import { useTrackerStore } from '../store/trackerStore'
import { useAuthStore } from '../store/authStore'
import { useRouter } from 'vue-router'

const trackerStore = useTrackerStore()
const authStore = useAuthStore()
const router = useRouter()

const csvFile = ref(null)
const csvInputRef = ref(null)
const unassignedMentorsPanel = ref([]) // Collapsed by default
const unassignedInternsPanel = ref([]) // Collapsed by default

// --- Role Options ---
const roleOptions = [
  { title: 'Intern / Member', value: 'member' },
  { title: 'Mentor', value: 'mentor' }
]
const csvRole = ref('member')

// --- Add Member Dialog ---
const addDialog = ref(false)
const addLoading = ref(false)
const addForm = ref(null)
const newMember = ref({ name: '', email: '', location: '', role: 'member' })

const checkAccess = () => {
  if (authStore.isAuthReady && authStore.userRole !== 'admin') {
    router.push('/')
  }
}

// Initialize store listener
onMounted(() => {
  checkAccess()
  trackerStore.subscribeEngineers()
  if (authStore.userRole === 'admin') {
    authStore.fetchAllUsers()
  }
})

watch(() => authStore.isAuthReady, checkAccess)

const getEngineersByTeam = (teamId) => {
  return trackerStore.engineers.filter(e => e.teamId === teamId && (e.role || 'member') !== 'mentor')
}

const getEngineersByTeamAndRole = (teamId, role) => {
  return trackerStore.engineers.filter(e => e.teamId === teamId && (e.role || 'member') === role)
}

const getUnassignedByRole = (role) => {
  return trackerStore.engineers.filter(e => e.teamId === null && (e.role || 'member') === role)
}

const onChange = (evt, newTeamId, role = 'member') => {
  if (evt.added) {
    trackerStore.updateEngineerTeam(evt.added.element.id, newTeamId, role)
  }
}

const deleteMember = (id) => {
  trackerStore.deleteEngineer(id)
}

const revertMember = (id) => {
  trackerStore.updateEngineerTeam(id, null, 'member')
}

const resetTeam = (teamId) => {
  const teamMembers = trackerStore.engineers.filter(e => e.teamId === teamId)
  teamMembers.forEach(e => trackerStore.updateEngineerTeam(e.id, null, 'member'))
}

const closeAddDialog = () => {
  addDialog.value = false
  newMember.value = { name: '', email: '', location: '', role: 'member' }
  if (addForm.value) addForm.value.reset()
}

const addMemberManually = async () => {
  const { valid } = await addForm.value.validate()
  if (!valid) return
  addLoading.value = true
  try {
    await trackerStore.addEngineersBulk([{
      name: newMember.value.name.trim(),
      email: newMember.value.email.trim(),
      location: newMember.value.location.trim(),
      role: newMember.value.role
    }])
    closeAddDialog()
  } finally {
    addLoading.value = false
  }
}

const handleFileUpload = (event) => {
  const file = event?.target?.files?.[0]
  if (!file) return;
  Papa.parse(file, {
    header: true,
    skipEmptyLines: true,
    complete: (results) => {
      const parsedData = results.data.map(row => ({
        name: row.Name || row.name || 'Unknown Engineer',
        location: row.Location || row.location || '',
        email: row.Email || row.email || '',
        role: csvRole.value
      }));
      if (parsedData.length > 0) {
        trackerStore.addEngineersBulk(parsedData)
      }
      if (csvInputRef.value) csvInputRef.value.value = '' // reset so same file can be re-selected
    }
  });
}

const getTeamColor = (id) => {
  const colors = {
    1: '#E65D24', // Logo Orange
    2: '#006DB6', // Logo Blue
    3: '#A9B400', // Logo Green
    4: '#5AB2E6', // Logo Light Blue
    5: '#F59E0B', // Amber
    6: '#EF4444', // Red
  }
  return colors[id] || '#9E9E9E'
}
</script>

<style scoped>
.min-h-100px {
  min-height: 100px;
}
.cursor-move {
  cursor: grab;
}
.cursor-move:active {
  cursor: grabbing;
}

/* Glass Panels & Columns */
.glass-panel {
  background: rgba(30, 41, 59, 0.6) !important;
  backdrop-filter: blur(12px);
  border: 1px solid rgba(255, 255, 255, 0.05);
}

.glass-column {
  background: rgba(15, 23, 42, 0.4) !important;
  backdrop-filter: blur(12px);
  border: 1px solid rgba(255, 255, 255, 0.03);
}

.drop-zone {
  border: 2px dashed rgba(255, 255, 255, 0.1) !important;
  transition: all 0.3s ease;
}

.drop-zone:hover {
  border-color: var(--team-color) !important;
  box-shadow: inset 0 0 30px var(--team-color) !important;
  background: rgba(15, 23, 42, 0.7) !important;
}

/* Engineer Pills */
.engineer-pill {
  background: rgba(30, 41, 59, 0.8) !important;
  border: 1px solid rgba(255, 255, 255, 0.08);
  position: relative;
  overflow: hidden;
  transition: all 0.2s ease;
}

.engineer-pill:hover {
  transform: translateY(-2px);
  border-color: rgba(255, 255, 255, 0.2);
  box-shadow: 0 8px 16px -4px rgba(0,0,0,0.4) !important;
}

.pill-glow {
  position: absolute;
  top: 0; left: 0; right: 0; bottom: 0;
  background: linear-gradient(90deg, transparent, rgba(255, 255, 255, 0.05));
  opacity: 0;
  transition: opacity 0.3s ease;
  pointer-events: none;
}

.engineer-pill:hover .pill-glow {
  opacity: 1;
}

.z-1 {
  z-index: 1;
}

.status-dot {
  width: 8px;
  height: 8px;
  border-radius: 50%;
}

.letter-spacing-1 {
  letter-spacing: 0.5px;
}

/* Scrollbars */
.list-container::-webkit-scrollbar {
  width: 6px;
}
.list-container::-webkit-scrollbar-track {
  background: transparent;
}
.list-container::-webkit-scrollbar-thumb {
  background-color: rgba(255, 255, 255, 0.1);
  border-radius: 10px;
}

.table-transparent {
  background: transparent !important;
}
.table-transparent th, .table-transparent td {
  border-bottom: 1px solid rgba(255,255,255,0.05) !important;
}
</style>

<style>
/* Light Theme Overrides for TeamComposition */
.v-theme--light .glass-panel {
  background: rgba(255, 255, 255, 0.7) !important;
  border-color: rgba(0, 0, 0, 0.1) !important;
}
.v-theme--light .glass-column {
  background: rgba(255, 255, 255, 0.4) !important;
  border-color: rgba(0, 0, 0, 0.05) !important;
}
.v-theme--light .drop-zone {
  border-color: rgba(0, 0, 0, 0.1) !important;
}
.v-theme--light .drop-zone:hover {
  background: rgba(255, 255, 255, 0.8) !important;
}
.v-theme--light .engineer-pill {
  background: rgba(255, 255, 255, 0.9) !important;
  border-color: rgba(0, 0, 0, 0.1) !important;
}
</style>
