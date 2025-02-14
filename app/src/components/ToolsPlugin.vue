<template>
    <section class="content">
        <div class="plugin-content">
            <p-header :title="pluginName">
                <p-button
                    :onClick="goBack"
                    slot="buttons"
                    type="clean back">
                    {{ $t('ui.backToTools') }}
                </p-button>

                <p-button
                    v-if="pluginHasConfig && hasPluginCustomOptions && pluginStandardOptionsVisible"
                    :onClick="showPluginCustomOptions"
                    slot="buttons"
                    type="clean back icon" 
                    icon="settings">
                    {{ $t('ui.goToPluginCustomOptions') }}
                </p-button>

                <p-button
                    v-if="pluginHasConfig && hasPluginCustomOptions && !pluginStandardOptionsVisible"
                    :onClick="showPluginStandardOptions"
                    slot="buttons"
                    type="clean back icon" 
                    icon="settings">
                    {{ $t('ui.goToPluginStandardOptions') }}
                </p-button>

                <p-button
                    v-if="pluginStandardOptionsVisible"
                    @click.native="saveSettings"
                    slot="buttons"
                    :disabled="buttonsLocked">
                    {{ $t('settings.saveSettings') }}
                </p-button>
            </p-header>

            <template v-if="!pluginHasConfig && !hasPluginCustomOptions">
                <p>{{ $t('toolsPlugin.thisPluginHasNoOptions') }}</p>
            </template>

            <div
                v-if="hasMessage"
                :class="'msg msg-icon msg-' + messageInOptions.type">
                <icon
                    :name="messageInOptions.type"
                    customWidth="28"
                    customHeight="28" />
                <div>
                    {{ messageInOptions.text }}
                </div>
            </div>

            <template v-if="pluginHasConfig && pluginStandardOptionsVisible">
                <fields-group
                    v-if="pluginSettingsDisplay === 'tabs'"
                    :title="pluginSettingsTabsLabel">
                    <tabs
                        ref="custom-settings-tabs"
                        id="custom-settings-tabs"
                        :items="settingsGroups">
                        <div
                            v-for="(groupName, index) of settingsGroups"
                            :slot="'tab-' + index"
                            :key="'tab-' + index">
                            <div>
                                <template v-for="(field, subindex) of getFieldsByGroupName(groupName)">
                                    <field
                                        v-if="checkDependencies(field.dependencies)"
                                        :label="getFieldLabel(field)"
                                        :key="'tab-' + index + '-field-' + subindex"
                                        :noLabelSpace="field.type === 'separator'"
                                        :labelFullWidth="field.type === 'wysiwyg'">
                                        <range-slider
                                            v-if="field.type === 'range'"
                                            :min="field.min"
                                            :max="field.max"
                                            :step="field.step"
                                            v-model="settingsValues[field.name]"
                                            :anchor="field.anchor"
                                            slot="field"></range-slider>

                                        <separator
                                            v-if="field.type === 'separator'"
                                            slot="field"
                                            :type="field.size"
                                            :label="field.label"
                                            :anchor="field.anchor"
                                            :note="field.note"></separator>

                                        <text-area
                                            v-if="field.type === 'textarea'"
                                            slot="field"
                                            :rows="field.rows"
                                            v-model="settingsValues[field.name]"
                                            :anchor="field.anchor"
                                            :spellcheck="$store.state.currentSite.config.spellchecking && field.spellcheck"
                                            :cols="field.cols"></text-area>

                                        <text-area
                                            v-if="field.type === 'wysiwyg'"
                                            slot="field"
                                            :id="'theme-settings-' + index"
                                            v-model="settingsValues[field.name]"
                                            :anchor="field.anchor"
                                            :wysiwyg="true"></text-area>

                                        <image-upload
                                            v-if="field.type === 'upload'"
                                            v-model="settingsValues[field.name]"
                                            slot="field"
                                            :anchor="field.anchor"
                                            imageType="pluginImages"
                                            :pluginDir="$route.params.pluginname"
                                            :addMediaFolderPath="false"
                                            :onBeforeRemove="removePluginImage"></image-upload>

                                        <small-image-upload
                                            v-if="field.type === 'smallupload'"
                                            v-model="settingsValues[field.name]"
                                            :anchor="field.anchor"
                                            imageType="pluginImages"
                                            :pluginDir="$route.params.pluginname"
                                            slot="field"
                                            :onBeforeRemove="removePluginImage"></small-image-upload>

                                        <radio-buttons
                                            v-if="field.type === 'radio'"
                                            :items="field.options"
                                            :name="field.name"
                                            v-model="settingsValues[field.name]"
                                            :anchor="field.anchor"
                                            slot="field" />

                                        <dropdown
                                            v-if="field.type === 'select'"
                                            slot="field"
                                            :multiple="field.multiple"
                                            v-model="settingsValues[field.name]"
                                            :id="field.anchor"
                                            :items="getDropdownOptions(field.options)"></dropdown>

                                        <switcher
                                            v-if="field.type === 'checkbox'"
                                            v-model="settingsValues[field.name]"
                                            :lower-zindex="true"
                                            :anchor="field.anchor"
                                            slot="field"></switcher>

                                        <color-picker
                                            v-if="field.type === 'colorpicker'"
                                            v-model="settingsValues[field.name]"
                                            :data-field="field.name"
                                            :anchor="field.anchor"
                                            slot="field"></color-picker>

                                        <posts-dropdown
                                            v-if="field.type === 'posts-dropdown'"
                                            v-model="settingsValues[field.name]"
                                            :allowed-post-status="field.allowedPostStatus || ['any']"
                                            :multiple="field.multiple"
                                            :anchor="field.anchor"
                                            slot="field"></posts-dropdown>

                                        <tags-dropdown
                                            v-if="field.type === 'tags-dropdown'"
                                            v-model="settingsValues[field.name]"
                                            :multiple="field.multiple"
                                            :anchor="field.anchor"
                                            slot="field"></tags-dropdown>

                                        <authors-dropdown
                                            v-if="field.type === 'authors-dropdown'"
                                            v-model="settingsValues[field.name]"
                                            :multiple="field.multiple"
                                            :anchor="field.anchor"
                                            slot="field"></authors-dropdown>

                                        <text-input
                                            v-if="isNormalInput(field.type)"
                                            slot="field"
                                            :type="field.type"
                                            :min="field.min"
                                            :max="field.max"
                                            :size="field.size"
                                            :step="field.step"
                                            :pattern="field.pattern"
                                            :spellcheck="$store.state.currentSite.config.spellchecking && field.spellcheck"
                                            v-model="settingsValues[field.name]"
                                            :anchor="field.anchor"
                                            :placeholder="field.placeholder"></text-input>

                                        <small
                                            v-if="field.note && field.type !== 'separator'"
                                            slot="note"
                                            class="note"
                                            v-pure-html="field.note">
                                        </small>
                                    </field>
                                </template>
                            </div>
                        </div>
                    </tabs>
                </fields-group>
                
                <template v-if="pluginSettingsDisplay === 'fieldsets'">
                    <fields-group
                        v-for="(groupName, index) of settingsGroups"
                        :key="'settings-group-' + index"
                        :title="groupName">
                        <template v-for="(field, subindex) of getFieldsByGroupName(groupName)">
                            <field
                                v-if="checkDependencies(field.dependencies)"
                                :label="getFieldLabel(field)"
                                :key="'tab-' + index + '-field-' + subindex"
                                :noLabelSpace="field.type === 'separator'"
                                :labelFullWidth="field.type === 'wysiwyg'">
                                <range-slider
                                    v-if="field.type === 'range'"
                                    :min="field.min"
                                    :max="field.max"
                                    :step="field.step"
                                    v-model="settingsValues[field.name]"
                                    :anchor="field.anchor"
                                    slot="field"></range-slider>

                                <separator
                                    v-if="field.type === 'separator'"
                                    slot="field"
                                    :type="field.size"
                                    :label="field.label"
                                    :anchor="field.anchor"
                                    :note="field.note"></separator>

                                <text-area
                                    v-if="field.type === 'textarea'"
                                    slot="field"
                                    :rows="field.rows"
                                    v-model="settingsValues[field.name]"
                                    :anchor="field.anchor"
                                    :spellcheck="$store.state.currentSite.config.spellchecking && field.spellcheck"
                                    :cols="field.cols"></text-area>

                                <text-area
                                    v-if="field.type === 'wysiwyg'"
                                    slot="field"
                                    :id="'theme-settings-' + index"
                                    v-model="settingsValues[field.name]"
                                    :anchor="field.anchor"
                                    :wysiwyg="true"></text-area>

                                <image-upload
                                    v-if="field.type === 'upload'"
                                    v-model="settingsValues[field.name]"
                                    slot="field"
                                    :anchor="field.anchor"
                                    imageType="pluginImages"
                                    :pluginDir="$route.params.pluginname"
                                    :addMediaFolderPath="false"
                                    :onBeforeRemove="removePluginImage"></image-upload>

                                <small-image-upload
                                    v-if="field.type === 'smallupload'"
                                    v-model="settingsValues[field.name]"
                                    :anchor="field.anchor"
                                    imageType="pluginImages"
                                    :pluginDir="$route.params.pluginname"
                                    slot="field"
                                    :onBeforeRemove="removePluginImage"></small-image-upload>

                                <radio-buttons
                                    v-if="field.type === 'radio'"
                                    :items="field.options"
                                    :name="field.name"
                                    v-model="settingsValues[field.name]"
                                    :anchor="field.anchor"
                                    slot="field" />

                                <dropdown
                                    v-if="field.type === 'select'"
                                    slot="field"
                                    :multiple="field.multiple"
                                    v-model="settingsValues[field.name]"
                                    :id="field.anchor"
                                    :items="getDropdownOptions(field.options)"></dropdown>

                                <switcher
                                    v-if="field.type === 'checkbox'"
                                    v-model="settingsValues[field.name]"
                                    :lower-zindex="true"
                                    :anchor="field.anchor"
                                    slot="field"></switcher>

                                <color-picker
                                    v-if="field.type === 'colorpicker'"
                                    v-model="settingsValues[field.name]"
                                    :data-field="field.name"
                                    :anchor="field.anchor"
                                    slot="field"></color-picker>

                                <posts-dropdown
                                    v-if="field.type === 'posts-dropdown'"
                                    v-model="settingsValues[field.name]"
                                    :allowed-post-status="field.allowedPostStatus || ['any']"
                                    :multiple="field.multiple"
                                    :anchor="field.anchor"
                                    slot="field"></posts-dropdown>

                                <tags-dropdown
                                    v-if="field.type === 'tags-dropdown'"
                                    v-model="settingsValues[field.name]"
                                    :multiple="field.multiple"
                                    :anchor="field.anchor"
                                    slot="field"></tags-dropdown>

                                <authors-dropdown
                                    v-if="field.type === 'authors-dropdown'"
                                    v-model="settingsValues[field.name]"
                                    :multiple="field.multiple"
                                    :anchor="field.anchor"
                                    slot="field"></authors-dropdown>

                                <text-input
                                    v-if="isNormalInput(field.type)"
                                    slot="field"
                                    :type="field.type"
                                    :min="field.min"
                                    :max="field.max"
                                    :size="field.size"
                                    :step="field.step"
                                    :pattern="field.pattern"
                                    :spellcheck="$store.state.currentSite.config.spellchecking && field.spellcheck"
                                    v-model="settingsValues[field.name]"
                                    :anchor="field.anchor"
                                    :placeholder="field.placeholder"></text-input>

                                <small
                                    v-if="field.note && field.type !== 'separator'"
                                    slot="note"
                                    class="note"
                                    v-pure-html="field.note">
                                </small>
                            </field>
                        </template>
                    </fields-group>
                </template>

                <p-footer>
                    <p-button
                        @click.native="saveSettings"
                        slot="buttons"
                        :disabled="buttonsLocked">
                        {{ $t('settings.saveSettings') }}
                    </p-button>
                </p-footer>
            </template>
            <template v-else>
                <iframe 
                    id="plugin-settings-root"
                    :src="this.pluginPath + '/options/index.html'"></iframe>
            </template>
        </div>
    </section>
</template>

<script>
import BackToTools from './mixins/BackToTools.js';
import Vue from 'vue';

export default {
    name: 'tools-plugin',
    mixins: [
        BackToTools
    ],
    computed: {
        settingsGroups () {
            let groups = [];

            this.settings.forEach(item => {
                if (groups.indexOf(item.group) === -1) {
                    groups.push(item.group);
                }
            });

            return groups;
        },
        pluginHasConfig () {
            return !!this.settings.length;
        }
    },
    data () {
        return {
            pluginName: '',
            pluginPath: '',
            settings: [],
            settingsValues: {},
            buttonsLocked: false,
            hasMessage: false,
            hasPluginCustomOptions: false,
            messageInOptions: null,
            pluginStandardOptionsVisible: true,
            pluginSettingsDisplay: 'fieldsets',
            pluginSettingsTabsLabel: ''
        };
    },
    async mounted () {
        this.loadPluginConfig(this.$route.params.pluginname, this.$route.params.name);
        document.getElementById('plugin-settings-root').addEventListener('load', async function () {
            this.contentWindow.window.document.querySelector('html').setAttribute('data-theme', await window.app.getCurrentAppTheme());
        }, false);
    },
    methods: {
        loadPluginConfig (pluginName, siteName) {
            mainProcessAPI.send('app-site-get-plugin-config', {
                siteName,
                pluginName
            });

            mainProcessAPI.receiveOnce('app-site-get-plugin-config-retrieved', result => {
                if (!result) {
                    this.$bus.$emit('alert-display', {
                        message: this.$t('tools.pluginLoadError'),
                        buttonStyle: 'danger'
                    });
                    return;
                }

                this.pluginName = result.pluginData.name;
                this.hasPluginCustomOptions = !!result.pluginData.usePluginSettingsView;
                this.hasMessage = !!result.pluginData.messageInOptions;
                this.messageInOptions = result.pluginData.messageInOptions;
                this.pluginSettingsDisplay = result.pluginData.settingsDisplay || 'fieldsets';
                this.pluginSettingsTabsLabel = result.pluginData.tabsTitle || this.$t('toolsPlugin.tabsLabel');

                if (this.hasPluginCustomOptions) {
                    this.pluginStandardOptionsVisible = false;
                    this.pluginPath = result.pluginData.path;
                }

                this.loadSettings(result.pluginData.config, result.pluginConfig);
            });
        },
        getFieldLabel (field) {
            if (field.type === 'separator') {
                return '';
            }

            if (field.label === '' || typeof field.label === 'undefined') {
                return ' ';
            }

            return field.label;
        },
        checkDependencies (dependencies) {
            if (!dependencies || !dependencies.length) {
                return true;
            }

            for (let i = 0; i < dependencies.length; i++) {
                let dependencyName = dependencies[i].field;
                let dependencyValue = dependencies[i].value;

                if (dependencyValue === "true" && this.settingsValues[dependencyName] !== true) {
                    return false;
                } else if (dependencyValue === "true") {
                    continue;
                }

                if (dependencyValue === "false" && this.settingsValues[dependencyName] !== false) {
                    return false;
                } else if (dependencyValue === "false") {
                    continue;
                }

                if (typeof dependencyValue === 'string' && dependencyValue.indexOf(',') > -1) {
                    let values = dependencyValue.split(',');

                    for (let i = 0; i < values.length; i++) {
                        if (this.settingsValues[dependencyName] === values[i]) {
                            continue;
                        }
                    }

                    return false;
                }

                if (dependencyValue !== this.settingsValues[dependencyName]) {
                    return false;
                }
            }

            return true;
        },
        isNormalInput (type) {
            return [
                'separator',
                'textarea',
                'wysiwyg',
                'radio',
                'select',
                'range',
                'upload',
                'smallupload',
                'checkbox',
                'colorpicker',
                'posts-dropdown',
                'authors-dropdown',
                'tags-dropdown'
            ].indexOf(type) === -1;
        },
        getDropdownOptions (inputOptions) {
            let options = {};

            for (let i = 0; i < inputOptions.length; i++) {
                options[inputOptions[i].value] = inputOptions[i].label;
            }

            return options;
        },
        getFieldsByGroupName (groupName) {
            return this.settings.filter(field => field.group === groupName);
        },
        loadSettings (config, savedConfig) {
            try {
                this.settings = JSON.parse(JSON.stringify(config));
                savedConfig = JSON.parse(savedConfig);
            } catch (e) {
                this.$bus.$emit('message-display', {
                    message: this.$t('toolsPlugin.pluginSettingsLoadError'),
                    type: 'warning',
                    lifeTime: 3
                });
                return;
            }

            let settings = config.map(field => {
                if (field.type !== 'separator') {
                    if (savedConfig && typeof savedConfig[field.name] !== 'undefined') {
                        return [field.name, savedConfig[field.name]];
                    }

                    return [field.name, field.value];
                }

                return false;
            });

            for (let setting of settings) {
                if (setting) {
                    Vue.set(this.settingsValues, setting[0], setting[1]);
                }
            }
        },
        saveSettings () {
            mainProcessAPI.send('app-site-save-plugin-config', {
                siteName: this.$route.params.name,
                pluginName: this.$route.params.pluginname,
                newConfig: this.settingsValues
            });

            mainProcessAPI.receiveOnce('app-site-plugin-config-saved', (result) => {
                if (result === true) {
                    this.$bus.$emit('message-display', {
                        message: this.$t('toolsPlugin.pluginSettingsSaveSuccess'),
                        type: 'success',
                        lifeTime: 3
                    });
                } else {
                    this.$bus.$emit('message-display', {
                        message: this.$t('toolsPlugin.pluginSettingsSaveError'),
                        type: 'warning',
                        lifeTime: 3
                    });
                }
            });
        },
        showPluginCustomOptions () {
            this.pluginStandardOptionsVisible = false;
        },
        showPluginStandardOptions () {
            this.pluginStandardOptionsVisible = true;
        },
        removePluginImage (filePath) {
            if (filePath) {
                mainProcessAPI.send('app-image-upload-remove', filePath, this.$route.params.name);
            }
        }
    }
}
</script>

<style lang="scss" scoped>
@import '../scss/variables.scss';
@import '../scss/notifications.scss';

#plugin-settings-root {
    border: none;
    height: calc(100vh - 200px);
    width: 100%;
}

.plugin-content {
    margin: 0 auto;
    max-width: $wrapper; 
    user-select: none;
}

.msg {
    background: var(--bg-secondary);
    margin-bottom: 3rem;
}
</style>
