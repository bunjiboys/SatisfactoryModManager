<template>
  <v-menu
    :close-on-content-click="false"
    offset-x
  >
    <template #activator="{ on, attrs }">
      <v-btn
        v-bind="attrs"
        class="my-2 px-2 d-inline-flex align-center"
        style="height: 28px"
        v-on="on"
      >
        <v-icon
          style="margin-right: 4px; font-size: 16px !important"
        >
          mdi-cog
        </v-icon>
        <span>Settings</span>
      </v-btn>
    </template>
    <v-card>
      <v-list class="menu">
        <v-menu
          :close-on-content-click="false"
          offset-x
          :nudge-right="20"
        >
          <template #activator="{ on }">
            <v-list-item
              v-on="on"
            >
              <v-list-item-action />
              <v-list-item-content>
                <v-list-item-title>Profiles</v-list-item-title>
              </v-list-item-content>
              <v-list-item-action>
                <v-icon>mdi-chevron-right</v-icon>
              </v-list-item-action>
            </v-list-item>
          </template>
          <v-card>
            <v-list class="menu">
              <v-list-item @click="exportProfile">
                <v-list-item-action>
                  <v-icon color="text">
                    mdi-content-save
                  </v-icon>
                </v-list-item-action>
                <v-list-item-content>
                  <v-list-item-title>Export profile</v-list-item-title>
                </v-list-item-content>
              </v-list-item>

              <v-divider
                class="custom"
                inset
              />

              <v-list-item @click="importProfileDialog = true">
                <v-list-item-action>
                  <v-icon color="text">
                    mdi-download
                  </v-icon>
                </v-list-item-action>
                <v-list-item-content>
                  <v-list-item-title>Import profile</v-list-item-title>
                </v-list-item-content>
                <v-dialog
                  v-model="importProfileDialog"
                >
                  <v-card>
                    <v-card-title>
                      Import profile
                    </v-card-title>
                    <v-card-text>
                      <v-form
                        ref="importProfileForm"
                        v-model="importProfileFormValid"
                        @update="importProfileFormValid = $event"
                        @submit.stop.prevent="importProfile"
                      >
                        <v-file-input
                          v-model="importProfileFile"
                          label="Profile file"
                          accept=".smmprofile"
                          required
                          :rules="[v => !!v || 'Choose a profile to import']"
                        />
                        <v-text-field
                          v-model="importProfileName"
                          label="Name"
                          required
                          :rules="[v => !!v || 'Profile name is required']"
                        />
                        <v-switch
                          v-model="importProfileVersions"
                          label="Import mod versions"
                        />
                        <span class="warning--text">{{ importProfileMessages.join('\n') }}</span>
                      </v-form>
                    </v-card-text>
                    <v-card-actions>
                      <v-btn
                        color="primary"
                        text
                        @click="importProfile"
                      >
                        Import
                      </v-btn>
                      <v-btn
                        color="primary"
                        text
                        @click="importProfileDialog = false"
                      >
                        Cancel
                      </v-btn>
                    </v-card-actions>
                  </v-card>
                </v-dialog>
              </v-list-item>

              <v-divider
                inset
                class="custom"
              />
            </v-list>
          </v-card>
        </v-menu>

        <v-divider
          v-if="installedSMLVersion"
          class="custom"
        />

        <v-menu
          v-if="installedSMLVersion"
          :close-on-content-click="false"
          offset-x
          :nudge-right="20"
        >
          <template #activator="{ on }">
            <v-list-item
              v-on="on"
            >
              <v-list-item-action />
              <v-list-item-content>
                <v-list-item-title>SML Versions</v-list-item-title>
              </v-list-item-content>
              <v-list-item-action>
                <v-icon>mdi-chevron-right</v-icon>
              </v-list-item-action>
            </v-list-item>
          </template>
          <v-list
            class="menu overflow-y-auto"
            style="max-height: 90vh"
          >
            <v-list-item
              @click="installSMLVersion('')"
            >
              <v-list-item-action>
                <v-icon v-if="!manifestSMLVersion">
                  mdi-check
                </v-icon>
              </v-list-item-action>
              <v-list-item-content>
                <v-list-item-title>Latest</v-list-item-title>
              </v-list-item-content>
            </v-list-item>
            <template
              v-for="(smlVersion, i) in smlVersions"
            >
              <v-divider
                :key="2 * i"
                class="custom"
              />
              <v-list-item
                :key="2 * i + 1"
                @click="installSMLVersion(smlVersion.version)"
              >
                <v-list-item-action>
                  <v-icon v-if="manifestSMLVersion && validAndEq(installedSMLVersion, smlVersion.version)">
                    mdi-check
                  </v-icon>
                </v-list-item-action>
                <v-list-item-content>
                  <v-list-item-title>{{ smlVersion.version }}</v-list-item-title>
                </v-list-item-content>
              </v-list-item>
            </template>
          </v-list>
        </v-menu>

        <v-divider
          class="custom"
        />

        <v-menu
          :close-on-content-click="false"
          offset-x
          :nudge-right="20"
        >
          <template #activator="{ on }">
            <v-list-item
              v-on="on"
            >
              <v-list-item-action />
              <v-list-item-content>
                <v-list-item-title>Debug</v-list-item-title>
              </v-list-item-content>
              <v-list-item-action>
                <v-icon>mdi-chevron-right</v-icon>
              </v-list-item-action>
            </v-list-item>
          </template>
          <v-card>
            <v-list class="menu">
              <v-list-item>
                <v-list-item-action />
                <v-list-item-content>
                  <v-list-item-title>Debug mode</v-list-item-title>
                </v-list-item-content>

                <v-list-item-action>
                  <v-switch
                    v-model="debugMode"
                  />
                </v-list-item-action>
              </v-list-item>

              <v-divider
                class="custom"
                inset
              />

              <v-list-item
                @click="clearCache"
              >
                <v-list-item-action />
                <v-list-item-content>
                  <v-list-item-title>Clear cache</v-list-item-title>
                </v-list-item-content>
              </v-list-item>

              <v-divider
                class="custom"
                inset
              />

              <v-list-item
                @click="exportDebugData"
              >
                <v-list-item-action />
                <v-list-item-content>
                  <v-list-item-title>Generate debug info</v-list-item-title>
                </v-list-item-content>
              </v-list-item>

              <v-divider
                inset
                class="custom"
              />
            </v-list>
          </v-card>
        </v-menu>

        <v-divider class="custom" />

        <v-list-item>
          <v-list-item-action>
            <v-icon color="text">
              mdi-cog
            </v-icon>
          </v-list-item-action>
          <v-list-item-content>
            <v-list-item-title>Settings</v-list-item-title>
          </v-list-item-content>
        </v-list-item>

        <v-divider class="custom" />

        <v-list-item>
          <v-list-item-action />
          <v-list-item-content>
            <v-list-item-title>Expand mod info on start</v-list-item-title>
          </v-list-item-content>

          <v-list-item-action>
            <v-switch
              v-model="expandModInfoOnStart"
            />
          </v-list-item-action>
        </v-list-item>

        <v-divider
          class="custom"
          inset
        />

        <v-list-item>
          <v-list-item-action />
          <v-list-item-content>
            <v-list-item-title>Disable download timeout</v-list-item-title>
          </v-list-item-content>

          <v-list-item-action>
            <v-switch
              v-model="disableDownloadTimeout"
            />
          </v-list-item-action>
        </v-list-item>

        <v-divider
          class="custom"
          inset
        />

        <v-list-item>
          <v-list-item-action>
            <v-icon color="text">
              mdi-information
            </v-icon>
          </v-list-item-action>
          <v-list-item-content>
            <v-list-item-title>About</v-list-item-title>
          </v-list-item-content>
        </v-list-item>

        <v-divider class="custom" />

        <v-list-item @click.stop="creditsDialog = true">
          <v-list-item-action />
          <v-list-item-content>
            <v-list-item-title>Credits</v-list-item-title>
          </v-list-item-content>

          <v-list-item-action>
            <v-icon color="text">
              mdi-information
            </v-icon>
          </v-list-item-action>
          <v-dialog v-model="creditsDialog">
            <v-card>
              <v-card-title>
                Credits
              </v-card-title>
              <v-card-text>
                Mircea Roata - programming<br>
                Deantendo - UI &amp; icons<br>
                Vilsol - <a
                  href="https://ficsit.app"
                  target="_blank"
                >ficsit.app</a>
              </v-card-text>
              <v-card-actions>
                <v-btn
                  color="primary"
                  text
                  @click="creditsDialog = false"
                >
                  Close
                </v-btn>
              </v-card-actions>
            </v-card>
          </v-dialog>
        </v-list-item>

        <v-divider
          class="custom"
          inset
        />

        <v-list-item @click.stop="attributionDialog = true">
          <v-list-item-action />
          <v-list-item-content>
            <v-list-item-title>Attribution</v-list-item-title>
          </v-list-item-content>

          <v-list-item-action>
            <v-icon color="text">
              mdi-information
            </v-icon>
          </v-list-item-action>
          <v-dialog v-model="attributionDialog">
            <v-card>
              <v-card-title>
                Attribution
              </v-card-title>
              <v-card-text>
                This app uses:<br>
                <a
                  href="https://github.com/electron/electron"
                  target="_blank"
                >Electron</a><br>
                <a
                  href="https://github.com/vuejs/vue"
                  target="_blank"
                >Vue</a><br>
                <a
                  href="https://github.com/vuetifyjs/vuetify"
                  target="_blank"
                >Vuetify</a><br>
                Licensed under MIT license<br>
                <a
                  href="https://materialdesignicons.com/"
                  target="_blank"
                >Material Design Icons</a><br>
                And many others
              </v-card-text>
              <v-card-actions>
                <v-btn
                  color="primary"
                  text
                  @click="attributionDialog = false"
                >
                  Close
                </v-btn>
              </v-card-actions>
            </v-card>
          </v-dialog>
        </v-list-item>

        <v-divider
          class="custom"
          inset
        />

        <v-divider
          class="custom"
          inset
        />

        <v-list-item>
          <v-list-item-action>
            <v-icon color="text">
              mdi-discord
            </v-icon>
          </v-list-item-action>
          <v-list-item-content>
            <v-list-item-title>Discord</v-list-item-title>
          </v-list-item-content>
        </v-list-item>

        <v-divider class="custom" />

        <v-list-item @click="moddingDiscord">
          <v-list-item-action />
          <v-list-item-content>
            <v-list-item-title>Satisfactory Modding</v-list-item-title>
          </v-list-item-content>

          <v-list-item-action>
            <v-icon color="text">
              mdi-open-in-new
            </v-icon>
          </v-list-item-action>
        </v-list-item>

        <v-divider
          class="custom"
          inset
        />

        <v-list-item @click="officialDiscord">
          <v-list-item-action />
          <v-list-item-content>
            <v-list-item-title>Official Satisfactory</v-list-item-title>
          </v-list-item-content>

          <v-list-item-action>
            <v-icon color="text">
              mdi-open-in-new
            </v-icon>
          </v-list-item-action>
        </v-list-item>

        <v-divider class="custom" />

        <template v-if="konami">
          <v-menu
            :close-on-content-click="false"
            offset-x
            :nudge-right="20"
          >
            <template #activator="{ on }">
              <v-list-item
                v-on="on"
              >
                <v-list-item-action />
                <v-list-item-content>
                  <v-list-item-title>Secret settings</v-list-item-title>
                </v-list-item-content>
                <v-list-item-action>
                  <v-icon>mdi-chevron-right</v-icon>
                </v-list-item-action>
              </v-list-item>
            </template>
            <v-card>
              <v-list class="menu">
                <v-list-item>
                  <v-list-item-action />
                  <v-list-item-content>
                    <v-list-item-title>Launch button</v-list-item-title>
                  </v-list-item-content>

                  <v-list-item-action>
                    <v-switch
                      v-model="launchButton"
                    />
                  </v-list-item-action>
                </v-list-item>
                <v-list-item>
                  <v-list-item-action />
                  <v-list-item-content>
                    <v-list-item-title>Launch Nyan</v-list-item-title>
                  </v-list-item-content>

                  <v-list-item-action>
                    <v-switch
                      v-model="launchCat"
                    />
                  </v-list-item-action>
                </v-list-item>
              </v-list>
            </v-card>
          </v-menu>

          <v-divider class="custom" />
        </template>

        <v-list-item>
          <v-list-item-action>
            <v-icon color="text">
              mdi-information
            </v-icon>
          </v-list-item-action>
          <v-list-item-content>
            <v-list-item-title>Satisfactory Mod Manager v{{ version }}</v-list-item-title>
          </v-list-item-content>
        </v-list-item>
      </v-list>
    </v-card>
  </v-menu>
</template>

<script>
import {
  clearCache, getLogFilePath, setDebug, validAndGreater, setTimeoutEnabled,
} from 'satisfactory-mod-manager-api';
import JSZip from 'jszip';
import fs from 'fs';
import path from 'path';
import { getCacheFolder } from 'platform-folders';
import StreamZip from 'node-stream-zip';
import gql from 'graphql-tag';
import { satisfies } from 'semver';
import { mapState } from 'vuex';
import { filenameFriendlyDate, filenamify, validAndEq } from '@/utils';
import { getSetting, saveSetting } from '~/settings';

/**
 * @param {JSZip} zip The zip file to add to
 * @param {string} filePath The file to add to the zip
 */
function addFileToZipIfExists(zip, filePath, customName) {
  if (fs.existsSync(filePath)) {
    zip.file(customName || path.basename(filePath), fs.createReadStream(filePath), { date: fs.statSync(filePath).mtime });
  }
}

export default {
  data() {
    return {
      attributionDialog: false,
      creditsDialog: false,
      importProfileFormValid: true,
      importProfileDialog: false,
      importProfileFile: null,
      importProfileMetadata: null,
      importProfileName: '',
      importProfileVersions: false,
      importProfileMessages: ['', ''],
      cachedDebugMode: false,
      cachedDisableDownloadTimeout: false,
      updateSMLVersionVal: 0,
    };
  },
  asyncComputed: {
    async version() {
      return this.$electron.ipcRenderer.invoke('getVersion');
    },
  },
  computed: {
    ...mapState([
      'konami',
    ]),
    expandModInfoOnStart: {
      get() {
        return this.$store.state.expandModInfoOnStart;
      },
      set(value) {
        this.$store.dispatch('setExpandModInfoOnStart', value);
      },
    },
    disableDownloadTimeout: {
      get() {
        return this.cachedDisableDownloadTimeout;
      },
      set(value) {
        setTimeoutEnabled(!value);
        saveSetting('disableDownloadTimeout', value);
        this.cachedDisableDownloadTimeout = value;
      },
    },
    debugMode: {
      get() {
        return this.cachedDebugMode;
      },
      set(value) {
        setDebug(value);
        saveSetting('debugMode', value);
        this.cachedDebugMode = value;
      },
    },
    launchButton: {
      get() {
        return this.$store.state.launchButton;
      },
      set(value) {
        this.$store.dispatch('launchButton', value);
      },
    },
    launchCat: {
      get() {
        return this.$store.state.launchCat;
      },
      set(value) {
        this.$store.dispatch('launchCat', value);
      },
    },
    installedSMLVersion() {
      return this.$store.state.installedMods.SML?.version;
    },
    manifestSMLVersion() {
      return this.$store.state.manifestItems.find((item) => item.id === 'SML')?.version;
    },
  },
  apollo: {
    smlVersions: {
      query: gql`
        query smlVersions {
          getSMLVersions(filter: {limit: 100}) {
            sml_versions {
              id,
              version,
            }
          }
        }
      `,
      update: (data) => data.getSMLVersions.sml_versions.filter((ver) => satisfies(ver.version, '>=2.0.0')),
    },
  },
  watch: {
    async importProfileFile(file) {
      if (file) {
        const zipData = new StreamZip({ file: file.path });
        try {
          await new Promise((resolve) => zipData.on('ready', resolve));
          const metadata = JSON.parse(zipData.entryDataSync('metadata.json').toString('utf8'));
          zipData.close();
          this.importProfileMetadata = metadata;
        } catch (e) {
          zipData.close();
          this.importProfileMetadata = null;
          this.$store.dispatch('showError', e);
        }
      } else {
        this.importProfileMetadata = null;
      }
    },
    importProfileMetadata(metadata) {
      if (metadata && validAndGreater(metadata.gameVersion, this.$store.state.selectedInstall.version)) {
        this.importProfileMessages[0] = `This profile is made for game version ${metadata.gameVersion}, but you're using an older version: ${this.$store.state.selectedInstall.version}. Things might not work as expected.`;
      } else {
        this.importProfileMessages[0] = '';
      }
    },
    importProfileName(name) {
      const validName = filenamify(name);
      if (name !== validName) {
        this.importProfileMessages[1] = `Profile will be saved as ${validName}`;
      } else {
        this.importProfileMessages[1] = '';
      }
    },
  },
  mounted() {
    this.cachedDebugMode = getSetting('debugMode', false);
    this.cachedDisableDownloadTimeout = getSetting('disableDownloadTimeout', false);

    if (this.debugMode) {
      setDebug(this.debugMode);
    }

    this.$root.$on('exportDebugData', this.exportDebugData);
    this.$root.$on('moddingDiscord', this.moddingDiscord);
    this.$root.$on('clearCache', this.clearCache);
  },
  methods: {
    moddingDiscord() {
      this.$electron.shell.openExternal('https://discord.gg/TShj39G');
    },
    officialDiscord() {
      this.$electron.shell.openExternal('https://discord.gg/Satisfactory');
    },
    clearCache() {
      clearCache();
      if (this.$store.state.selectedInstall) {
        this.$store.state.selectedInstall.clearCache();
      }
    },
    async exportDebugData() {
      const result = await this.$electron.ipcRenderer.invoke('saveDialog', {
        title: 'Save debug data as',
        filters: [
          { name: 'SMM Debug Data', extensions: ['zip'] },
        ],
        defaultPath: `SMMDebug_${filenameFriendlyDate(new Date())}.zip`,
      });

      if (result) {
        const debugDataZip = new JSZip();
        const metadata = {
          installsFound: this.$store.state.satisfactoryInstalls,
          selectedInstall: this.$store.state.selectedInstall,
          profiles: this.$store.state.profiles,
          selectedProfile: this.$store.state.selectedProfile,
          installedMods: this.$store.state.selectedInstall?.mods,
          smlVersion: this.$store.state.selectedInstall?.smlVersion,
          bootstrapperVersion: this.$store.state.selectedInstall?.bootstrapperVersion,
          smmVersion: this.version,
          modsEnabled: this.$store.state.modsEnabled,
        };
        debugDataZip.file('metadata.json', JSON.stringify(metadata, null, 4));
        addFileToZipIfExists(debugDataZip, getLogFilePath(), 'SatisfactoryModManager.log');
        addFileToZipIfExists(debugDataZip, path.join(getCacheFolder(), 'FactoryGame', 'Saved', 'Logs', 'FactoryGame.log'));

        try {
          await new Promise((resolve, reject) => {
            debugDataZip.generateNodeStream().pipe(fs.createWriteStream(result)).on('finish', resolve).on('error', reject);
          });
        } catch (e) {
          this.$store.dispatch('showError', e);
        }
      }
    },
    async importProfile() {
      if (this.$refs.importProfileForm.validate()) {
        this.importProfileName = filenamify(this.importProfileName);
        const importProfileProgress = {
          id: '__importProfile__',
          progresses: [{
            id: '', progress: -1, message: `Importing profile as ${this.importProfileName}`, fast: false,
          }],
        };
        this.$store.state.inProgress.push(importProfileProgress);
        try {
          await this.$store.state.selectedInstall.importProfile(this.importProfileFile.path, this.importProfileName, this.importProfileVersions);
          this.$store.state.inProgress.remove(importProfileProgress);
          const newProfile = { name: this.importProfileName, items: [] }; // TODO: Items
          this.$store.state.profiles.push(newProfile);
          await this.$store.dispatch('selectProfile', newProfile);
        } catch (e) {
          this.$store.dispatch('showError', e);
          this.$store.state.inProgress.remove(importProfileProgress);
        }
        this.importProfileFile = null;
        this.importProfileName = '';
        this.importProfileVersions = false;
        this.importProfileDialog = false;
      }
    },
    async exportProfile() {
      const result = await this.$electron.ipcRenderer.invoke('saveDialog', {
        title: 'Export profile as',
        filters: [
          { name: 'SMM Profile', extensions: ['smmprofile'] },
        ],
        defaultPath: `${this.$store.state.selectedProfile.name}.smmprofile`,
      });
      if (result) {
        const exportProfileProgress = {
          id: '__exportProfile__',
          progresses: [{
            id: '', progress: -1, message: `Exporting profile ${this.$store.state.selectedProfile.name}`, fast: false,
          }],
        };
        this.$store.state.inProgress.push(exportProfileProgress);
        try {
          await this.$store.state.selectedInstall.exportProfile(result);
        } catch (e) {
          this.$store.dispatch('showError', e);
        }
        this.$store.state.inProgress.remove(exportProfileProgress);
      }
    },
    async installSMLVersion(version) {
      await this.$store.dispatch('installSMLVersion', version);
      this.updateSMLVersionVal += 1;
    },
    validAndEq,
  },
};
</script>
