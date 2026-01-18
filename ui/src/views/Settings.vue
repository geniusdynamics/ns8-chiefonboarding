<!--
  Copyright (C) 2022 Nethesis S.r.l.
  SPDX-License-Identifier: GPL-3.0-or-later
-->
<template>
  <cv-grid fullWidth>
    <cv-row>
      <cv-column class="page-title">
        <h2>{{ $t("settings.title") }}</h2>
      </cv-column>
    </cv-row>
    <cv-row v-if="error.getConfiguration">
      <cv-column>
        <NsInlineNotification
          kind="error"
          :title="$t('action.get-configuration')"
          :description="error.getConfiguration"
          :showCloseButton="false"
        />
      </cv-column>
    </cv-row>
    <cv-row>
      <cv-column>
        <cv-tile light>
          <cv-form @submit.prevent="configureModule">
            <cv-text-input
              :label="$t('settings.chiefonboarding_fqdn')"
              placeholder="chiefonboarding.example.org"
              v-model.trim="host"
              class="mg-bottom"
              :invalid-message="$t(error.host)"
              :disabled="loading.getConfiguration || loading.configureModule"
              ref="host"
            >
            </cv-text-input>
            <cv-toggle
              value="letsEncrypt"
              :label="$t('settings.lets_encrypt')"
              v-model="isLetsEncryptEnabled"
              :disabled="loading.getConfiguration || loading.configureModule"
              class="mg-bottom"
            >
              <template slot="text-left">{{
                $t("settings.disabled")
              }}</template>
              <template slot="text-right">{{
                $t("settings.enabled")
              }}</template>
            </cv-toggle>
            <cv-toggle
              value="httpToHttps"
              :label="$t('settings.http_to_https')"
              v-model="isHttpToHttpsEnabled"
              :disabled="loading.getConfiguration || loading.configureModule"
              class="mg-bottom"
            >
              <template slot="text-left">{{
                $t("settings.disabled")
              }}</template>
              <template slot="text-right">{{
                $t("settings.enabled")
              }}</template>
            </cv-toggle>
              <!-- advanced options -->
            <cv-accordion ref="accordion" class="maxwidth mg-bottom">
              <cv-accordion-item :open="toggleAccordion[0]">
                <template slot="title">{{ $t("settings.advanced") }}</template>
                <template slot="content">
                  <cv-toggle
                    value="allowGoogleSso"
                    :label="$t('settings.allow_google_sso')"
                    v-model="isAllowGoogleSsoEnabled"
                    :disabled="loading.getConfiguration || loading.configureModule"
                    class="mg-bottom"
                  >
                    <template slot="text-left">{{
                      $t("settings.disabled")
                    }}</template>
                    <template slot="text-right">{{
                      $t("settings.enabled")
                    }}</template>
                  </cv-toggle>
                  <cv-text-input
                    :label="$t('settings.google_sso_client_id')"
                    placeholder="xxxxx"
                    v-model.trim="googleSsoClientId"
                    class="mg-bottom"
                    :invalid-message="$t(error.google_sso_client_id)"
                    :disabled="loading.getConfiguration || loading.configureModule"
                    ref="googleSsoClientId"
                  >
                  </cv-text-input>
                  <cv-text-input
                    :label="$t('settings.google_sso_secret')"
                    placeholder="xxxxx"
                    v-model.trim="googleSsoSecret"
                    class="mg-bottom"
                    :invalid-message="$t(error.google_sso_secret)"
                    :disabled="loading.getConfiguration || loading.configureModule"
                    ref="googleSsoSecret"
                  >
                  </cv-text-input>
                  <cv-toggle
                    value="ssoAutoCreateUser"
                    :label="$t('settings.sso_auto_create_user')"
                    v-model="isSsoAutoCreateUserEnabled"
                    :disabled="loading.getConfiguration || loading.configureModule"
                    class="mg-bottom"
                  >
                    <template slot="text-left">{{
                      $t("settings.disabled")
                    }}</template>
                    <template slot="text-right">{{
                      $t("settings.enabled")
                    }}</template>
                  </cv-toggle>
                  <cv-text-input
                    :label="$t('settings.oidc_role_new_hire_pattern')"
                    placeholder="^cn=Newhires.*"
                    v-model.trim="oidcRoleNewHirePattern"
                    class="mg-bottom"
                    :invalid-message="$t(error.oidc_role_new_hire_pattern)"
                    :disabled="loading.getConfiguration || loading.configureModule"
                    ref="oidcRoleNewHirePattern"
                  >
                  </cv-text-input>
                  <cv-text-input
                    :label="$t('settings.oidc_role_admin_pattern')"
                    placeholder="^cn=Administrators.*"
                    v-model.trim="oidcRoleAdminPattern"
                    class="mg-bottom"
                    :invalid-message="$t(error.oidc_role_admin_pattern)"
                    :disabled="loading.getConfiguration || loading.configureModule"
                    ref="oidcRoleAdminPattern"
                  >
                  </cv-text-input>
                  <cv-text-input
                    :label="$t('settings.oidc_role_manager_pattern')"
                    placeholder="^cn=Managers.*"
                    v-model.trim="oidcRoleManagerPattern"
                    class="mg-bottom"
                    :invalid-message="$t(error.oidc_role_manager_pattern)"
                    :disabled="loading.getConfiguration || loading.configureModule"
                    ref="oidcRoleManagerPattern"
                  >
                  </cv-text-input>
                  <cv-text-input
                    :label="$t('settings.oidc_role_path_in_return')"
                    placeholder="groups"
                    v-model.trim="oidcRolePathInReturn"
                    class="mg-bottom"
                    :invalid-message="$t(error.oidc_role_path_in_return)"
                    :disabled="loading.getConfiguration || loading.configureModule"
                    ref="oidcRolePathInReturn"
                  >
                  </cv-text-input>
                  <cv-text-input
                    :label="$t('settings.allauth_providers')"
                    placeholder="openid_connect"
                    v-model.trim="allauthProviders"
                    class="mg-bottom"
                    :invalid-message="$t(error.allauth_providers)"
                    :disabled="loading.getConfiguration || loading.configureModule"
                    ref="allauthProviders"
                  >
                  </cv-text-input>
                </template>
              </cv-accordion-item>
            </cv-accordion>
            <cv-row v-if="error.configureModule">
              <cv-column>
                <NsInlineNotification
                  kind="error"
                  :title="$t('action.configure-module')"
                  :description="error.configureModule"
                  :showCloseButton="false"
                />
              </cv-column>
            </cv-row>
            <NsButton
              kind="primary"
              :icon="Save20"
              :loading="loading.configureModule"
              :disabled="loading.getConfiguration || loading.configureModule"
              >{{ $t("settings.save") }}</NsButton
            >
          </cv-form>
        </cv-tile>
      </cv-column>
    </cv-row>
  </cv-grid>
</template>

<script>
import to from "await-to-js";
import { mapState } from "vuex";
import {
  QueryParamService,
  UtilService,
  TaskService,
  IconService,
  PageTitleService,
} from "@nethserver/ns8-ui-lib";

export default {
  name: "Settings",
  mixins: [
    TaskService,
    IconService,
    UtilService,
    QueryParamService,
    PageTitleService,
  ],
  pageTitle() {
    return this.$t("settings.title") + " - " + this.appName;
  },
  data() {
    return {
      q: {
        page: "settings",
      },
      urlCheckInterval: null,
      host: "",
      isLetsEncryptEnabled: false,
      isHttpToHttpsEnabled: true,
      isAllowGoogleSsoEnabled: false,
      googleSsoClientId: "",
      googleSsoSecret: "",
      isSsoAutoCreateUserEnabled: true,
      oidcRoleNewHirePattern: "",
      oidcRoleAdminPattern: "",
      oidcRoleManagerPattern: "",
      oidcRolePathInReturn: "groups",
      allauthProviders: "",
      loading: {
        getConfiguration: false,
        configureModule: false,
      },
      error: {
        getConfiguration: "",
        configureModule: "",
        host: "",
        lets_encrypt: "",
        http2https: "",
        allow_google_sso: "",
        google_sso_client_id: "",
        google_sso_secret: "",
        sso_auto_create_user: "",
        oidc_role_new_hire_pattern: "",
        oidc_role_admin_pattern: "",
        oidc_role_manager_pattern: "",
        oidc_role_path_in_return: "",
        allauth_providers: "",
      },
    };
  },
  computed: {
    ...mapState(["instanceName", "core", "appName"]),
  },
  created() {
    this.getConfiguration();
  },
  beforeRouteEnter(to, from, next) {
    next((vm) => {
      vm.watchQueryData(vm);
      vm.urlCheckInterval = vm.initUrlBindingForApp(vm, vm.q.page);
    });
  },
  beforeRouteLeave(to, from, next) {
    clearInterval(this.urlCheckInterval);
    next();
  },
  methods: {
    async getConfiguration() {
      this.loading.getConfiguration = true;
      this.error.getConfiguration = "";
      const taskAction = "get-configuration";
      const eventId = this.getUuid();

      // register to task error
      this.core.$root.$once(
        `${taskAction}-aborted-${eventId}`,
        this.getConfigurationAborted
      );

      // register to task completion
      this.core.$root.$once(
        `${taskAction}-completed-${eventId}`,
        this.getConfigurationCompleted
      );

      const res = await to(
        this.createModuleTaskForApp(this.instanceName, {
          action: taskAction,
          extra: {
            title: this.$t("action." + taskAction),
            isNotificationHidden: true,
            eventId,
          },
        })
      );
      const err = res[0];

      if (err) {
        console.error(`error creating task ${taskAction}`, err);
        this.error.getConfiguration = this.getErrorMessage(err);
        this.loading.getConfiguration = false;
        return;
      }
    },
    getConfigurationAborted(taskResult, taskContext) {
      console.error(`${taskContext.action} aborted`, taskResult);
      this.error.getConfiguration = this.$t("error.generic_error");
      this.loading.getConfiguration = false;
    },
    getConfigurationCompleted(taskContext, taskResult) {
      const config = taskResult.output;
      this.host = config.host;
      this.isLetsEncryptEnabled = config.lets_encrypt;
      this.isHttpToHttpsEnabled = config.http2https;
      this.isAllowGoogleSsoEnabled = config.allow_google_sso;
      this.googleSsoClientId = config.google_sso_client_id;
      this.googleSsoSecret = config.google_sso_secret;
      this.isSsoAutoCreateUserEnabled = config.sso_auto_create_user;
      this.oidcRoleNewHirePattern = config.oidc_role_new_hire_pattern;
      this.oidcRoleAdminPattern = config.oidc_role_admin_pattern;
      this.oidcRoleManagerPattern = config.oidc_role_manager_pattern;
      this.oidcRolePathInReturn = config.oidc_role_path_in_return;
      this.allauthProviders = config.allauth_providers;

      this.loading.getConfiguration = false;
      this.focusElement("host");
    },
    validateConfigureModule() {
      this.clearErrors(this);

      let isValidationOk = true;
      if (!this.host) {
        this.error.host = "common.required";

        if (isValidationOk) {
          this.focusElement("host");
        }
        isValidationOk = false;
      }
      return isValidationOk;
    },
    configureModuleValidationFailed(validationErrors) {
      this.loading.configureModule = false;
      let focusAlreadySet = false;

      for (const validationError of validationErrors) {
        const param = validationError.parameter;
        // set i18n error message
        this.error[param] = this.$t("settings." + validationError.error);

        if (!focusAlreadySet) {
          this.focusElement(param);
          focusAlreadySet = true;
        }
      }
    },
    async configureModule() {
      this.error.test_imap = false;
      this.error.test_smtp = false;
      const isValidationOk = this.validateConfigureModule();
      if (!isValidationOk) {
        return;
      }

      this.loading.configureModule = true;
      const taskAction = "configure-module";
      const eventId = this.getUuid();

      // register to task error
      this.core.$root.$once(
        `${taskAction}-aborted-${eventId}`,
        this.configureModuleAborted
      );

      // register to task validation
      this.core.$root.$once(
        `${taskAction}-validation-failed-${eventId}`,
        this.configureModuleValidationFailed
      );

      // register to task completion
      this.core.$root.$once(
        `${taskAction}-completed-${eventId}`,
        this.configureModuleCompleted
      );
      const res = await to(
        this.createModuleTaskForApp(this.instanceName, {
          action: taskAction,
          data: {
            host: this.host,
            lets_encrypt: this.isLetsEncryptEnabled,
            http2https: this.isHttpToHttpsEnabled,
            allow_google_sso: this.isAllowGoogleSsoEnabled,
            google_sso_client_id: this.googleSsoClientId,
            google_sso_secret: this.googleSsoSecret,
            sso_auto_create_user: this.isSsoAutoCreateUserEnabled,
            oidc_role_new_hire_pattern: this.oidcRoleNewHirePattern,
            oidc_role_admin_pattern: this.oidcRoleAdminPattern,
            oidc_role_manager_pattern: this.oidcRoleManagerPattern,
            oidc_role_path_in_return: this.oidcRolePathInReturn,
            allauth_providers: this.allauthProviders,
          },
          extra: {
            title: this.$t("settings.instance_configuration", {
              instance: this.instanceName,
            }),
            description: this.$t("settings.configuring"),
            eventId,
          },
        })
      );
      const err = res[0];

      if (err) {
        console.error(`error creating task ${taskAction}`, err);
        this.error.configureModule = this.getErrorMessage(err);
        this.loading.configureModule = false;
        return;
      }
    },
    configureModuleAborted(taskResult, taskContext) {
      console.error(`${taskContext.action} aborted`, taskResult);
      this.error.configureModule = this.$t("error.generic_error");
      this.loading.configureModule = false;
    },
    configureModuleCompleted() {
      this.loading.configureModule = false;

      // reload configuration
      this.getConfiguration();
    },
  },
};
</script>

<style scoped lang="scss">
@import "../styles/carbon-utils";
.mg-bottom {
  margin-bottom: $spacing-06;
}

.maxwidth {
  max-width: 38rem;
}
</style>