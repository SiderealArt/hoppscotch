<template>
  <SmartModal v-if="show" :title="t('team.invite')" @close="hideModal">
    <template #body>
      <div v-if="sendInvitesResult.length" class="flex flex-col px-4">
        <div class="flex flex-col max-w-md justify-center items-center">
          <SmartIcon class="h-6 text-accent w-6" name="users" />
          <h3 class="my-2 text-center text-lg">
            {{ t("team.we_sent_invite_link") }}
          </h3>
          <p class="text-center">
            {{ t("team.we_sent_invite_link_description") }}
          </p>
        </div>
        <div
          class="
            flex
            border border-dividerLight
            mt-8
            rounded
            flex-col
            space-y-6
            p-4
          "
        >
          <div
            v-for="(invitee, index) in sendInvitesResult"
            :key="`invitee-${index}`"
          >
            <p class="flex items-center">
              <i
                class="material-icons mr-4"
                :class="
                  invitee.status === 'error' ? 'text-red-500' : 'text-green-500'
                "
              >
                {{
                  invitee.status === "error"
                    ? "error_outline"
                    : "mark_email_read"
                }}
              </i>
              <span class="truncate">{{ invitee.email }}</span>
            </p>
            <p v-if="invitee.status === 'error'" class="ml-8 text-red-500 mt-2">
              {{ getErrorMessage(invitee.error) }}
            </p>
          </div>
        </div>
      </div>
      <div
        v-else-if="sendingInvites"
        class="flex p-4 items-center justify-center"
      >
        <SmartSpinner />
      </div>
      <div v-else class="flex flex-col px-2">
        <div class="flex flex-1 justify-between items-center">
          <label for="memberList" class="pb-4 px-4">
            {{ t("team.pending_invites") }}
          </label>
        </div>
        <div class="divide-y divide-dividerLight border-divider border rounded">
          <div
            v-if="pendingInvites.loading"
            class="flex p-4 items-center justify-center"
          >
            <SmartSpinner />
          </div>
          <div v-else>
            <div
              v-if="!pendingInvites.loading && E.isRight(pendingInvites.data)"
            >
              <div
                v-for="(invitee, index) in pendingInvites.data.right.team
                  .teamInvitations"
                :key="`invitee-${index}`"
                class="divide-x divide-dividerLight flex"
              >
                <input
                  v-if="invitee"
                  class="
                    bg-transparent
                    flex flex-1
                    text-secondaryLight
                    py-2
                    px-4
                  "
                  :placeholder="`${t('team.email')}`"
                  :name="'param' + index"
                  :value="invitee.inviteeEmail"
                  readonly
                />
                <input
                  class="
                    bg-transparent
                    flex flex-1
                    text-secondaryLight
                    py-2
                    px-4
                  "
                  :placeholder="`${t('team.permissions')}`"
                  :name="'value' + index"
                  :value="
                    typeof invitee.inviteeRole === 'string'
                      ? invitee.inviteeRole
                      : JSON.stringify(invitee.inviteeRole)
                  "
                  readonly
                />
                <div class="flex">
                  <ButtonSecondary
                    v-tippy="{ theme: 'tooltip' }"
                    :title="t('action.remove')"
                    svg="trash"
                    color="red"
                    @click.native="removeInvitee(invitee.id)"
                  />
                </div>
              </div>
            </div>
            <div
              v-if="
                E.isRight(pendingInvites.data) &&
                pendingInvites.data.right.team.teamInvitations.length === 0
              "
              class="
                flex flex-col
                text-secondaryLight
                p-4
                items-center
                justify-center
              "
            >
              <span class="text-center">
                {{ t("empty.pending_invites") }}
              </span>
            </div>
            <div
              v-if="!pendingInvites.loading && E.isLeft(pendingInvites.data)"
              class="flex flex-col p-4 items-center"
            >
              <i class="mb-4 material-icons">help_outline</i>
              {{ t("error.something_went_wrong") }}
            </div>
          </div>
        </div>
        <div class="flex pt-4 flex-1 justify-between items-center">
          <label for="memberList" class="p-4">
            {{ t("team.invite_tooltip") }}
          </label>
          <div class="flex">
            <ButtonSecondary
              svg="plus"
              :label="t('add.new')"
              filled
              @click.native="addNewInvitee"
            />
          </div>
        </div>
        <div class="divide-y divide-dividerLight border-divider border rounded">
          <div
            v-for="(invitee, index) in newInvites"
            :key="`new-invitee-${index}`"
            class="divide-x divide-dividerLight flex"
          >
            <input
              v-model="invitee.key"
              class="bg-transparent flex flex-1 py-2 px-4"
              :placeholder="`${t('team.email')}`"
              :name="'invitee' + index"
              autofocus
            />
            <span>
              <tippy
                ref="newInviteeOptions"
                interactive
                trigger="click"
                theme="popover"
                arrow
              >
                <template #trigger>
                  <span class="select-wrapper">
                    <input
                      class="
                        bg-transparent
                        cursor-pointer
                        flex flex-1
                        py-2
                        px-4
                      "
                      :placeholder="`${t('team.permissions')}`"
                      :name="'value' + index"
                      :value="
                        typeof invitee.value === 'string'
                          ? invitee.value
                          : JSON.stringify(invitee.value)
                      "
                      readonly
                    />
                  </span>
                </template>
                <SmartItem
                  label="OWNER"
                  @click.native="
                    () => {
                      updateNewInviteeRole(index, 'OWNER')
                      newInviteeOptions[index].tippy().hide()
                    }
                  "
                />
                <SmartItem
                  label="EDITOR"
                  @click.native="
                    () => {
                      updateNewInviteeRole(index, 'EDITOR')
                      newInviteeOptions[index].tippy().hide()
                    }
                  "
                />
                <SmartItem
                  label="VIEWER"
                  @click.native="
                    () => {
                      updateNewInviteeRole(index, 'VIEWER')
                      newInviteeOptions[index].tippy().hide()
                    }
                  "
                />
              </tippy>
            </span>
            <div class="flex">
              <ButtonSecondary
                id="member"
                v-tippy="{ theme: 'tooltip' }"
                :title="t('action.remove')"
                svg="trash"
                color="red"
                @click.native="removeNewInvitee(index)"
              />
            </div>
          </div>
          <div
            v-if="newInvites.length === 0"
            class="
              flex flex-col
              text-secondaryLight
              p-4
              items-center
              justify-center
            "
          >
            <img
              :src="`/images/states/${$colorMode.value}/add_group.svg`"
              loading="lazy"
              class="
                flex-col
                mb-4
                object-contain object-center
                h-16
                w-16
                inline-flex
              "
              :alt="`${t('empty.invites')}`"
            />
            <span class="text-center pb-4">
              {{ t("empty.invites") }}
            </span>
            <ButtonSecondary
              :label="t('add.new')"
              filled
              @click.native="addNewInvitee"
            />
          </div>
        </div>
        <div
          v-if="newInvites.length"
          class="
            px-4
            mt-4
            py-4
            rounded
            border border-dividerLight
            flex flex-col
            items-start
          "
        >
          <span
            class="
              mb-4
              px-2
              py-1
              flex
              justify-center
              items-center
              font-semibold
              rounded-full
              bg-primaryDark
              border border-divider
            "
          >
            <i class="text-secondaryLight mr-2 material-icons">help_outline</i>
            {{ t("profile.roles") }}
          </span>
          <p>
            <span class="text-secondaryLight">
              {{ t("profile.roles_description") }}
            </span>
          </p>
          <ul class="mt-4 space-y-4">
            <li class="flex">
              <span
                class="
                  font-semibold
                  text-secondaryDark
                  uppercase
                  truncate
                  max-w-16
                  w-1/4
                "
              >
                {{ t("profile.owner") }}
              </span>
              <span class="flex flex-1">
                {{ t("profile.owner_description") }}
              </span>
            </li>
            <li class="flex">
              <span
                class="
                  font-semibold
                  text-secondaryDark
                  uppercase
                  truncate
                  max-w-16
                  w-1/4
                "
              >
                {{ t("profile.editor") }}
              </span>
              <span class="flex flex-1">
                {{ t("profile.editor_description") }}
              </span>
            </li>
            <li class="flex">
              <span
                class="
                  font-semibold
                  text-secondaryDark
                  uppercase
                  truncate
                  max-w-16
                  w-1/4
                "
              >
                {{ t("profile.viewer") }}
              </span>
              <span class="flex flex-1">
                {{ t("profile.viewer_description") }}
              </span>
            </li>
          </ul>
        </div>
      </div>
    </template>
    <template #footer>
      <p
        v-if="sendInvitesResult.length"
        class="flex flex-1 text-secondaryLight justify-between"
      >
        <SmartAnchor
          class="link"
          :label="`← \xA0 ${t('team.invite_more')}`"
          @click.native="
            () => {
              sendInvitesResult = []
              newInvites = [
                {
                  key: '',
                  value: 'VIEWRER',
                },
              ]
            }
          "
        />
        <SmartAnchor
          class="link"
          :label="`${t('action.dismiss')}`"
          @click.native="hideModal"
        />
      </p>
      <span v-else>
        <ButtonPrimary :label="t('team.invite')" @click.native="sendInvites" />
        <ButtonSecondary
          :label="t('action.cancel')"
          @click.native="hideModal"
        />
      </span>
    </template>
  </SmartModal>
</template>

<script setup lang="ts">
import { watch, ref, reactive, computed } from "@nuxtjs/composition-api"
import * as T from "fp-ts/Task"
import * as E from "fp-ts/Either"
import * as A from "fp-ts/Array"
import * as O from "fp-ts/Option"
import { flow, pipe } from "fp-ts/function"
import { Email, EmailCodec } from "../../helpers/backend/types/Email"
import {
  TeamInvitationAddedDocument,
  TeamInvitationRemovedDocument,
  TeamMemberRole,
  GetPendingInvitesDocument,
  GetPendingInvitesQuery,
  GetPendingInvitesQueryVariables,
} from "../../helpers/backend/graphql"
import {
  createTeamInvitation,
  CreateTeamInvitationErrors,
  revokeTeamInvitation,
} from "../../helpers/backend/mutations/TeamInvitation"
import { GQLError, useGQLQuery } from "~/helpers/backend/GQLClient"
import { useI18n, useToast } from "~/helpers/utils/composables"

const t = useI18n()

const toast = useToast()

const newInviteeOptions = ref<any | null>(null)

const props = defineProps({
  show: Boolean,
  editingTeamID: { type: String, default: null },
})

const emit = defineEmits<{
  (e: "hide-modal"): void
}>()

const pendingInvites = useGQLQuery<
  GetPendingInvitesQuery,
  GetPendingInvitesQueryVariables,
  ""
>({
  query: GetPendingInvitesDocument,
  variables: reactive({
    teamID: props.editingTeamID,
  }),
  updateSubs: computed(() =>
    !props.editingTeamID
      ? []
      : [
          {
            key: 4,
            query: TeamInvitationAddedDocument,
            variables: {
              teamID: props.editingTeamID,
            },
          },
          {
            key: 5,
            query: TeamInvitationRemovedDocument,
            variables: {
              teamID: props.editingTeamID,
            },
          },
        ]
  ),
  defer: true,
})

watch(
  () => props.editingTeamID,
  () => {
    if (props.editingTeamID) {
      pendingInvites.execute({
        teamID: props.editingTeamID,
      })
    }
  }
)

const removeInvitee = async (id: string) => {
  const result = await revokeTeamInvitation(id)()
  if (E.isLeft(result)) {
    toast.error(`${t("error.something_went_wrong")}`)
  } else {
    toast.success(`${t("team.member_removed")}`)
  }
}

const newInvites = ref<Array<{ key: string; value: TeamMemberRole }>>([
  {
    key: "",
    value: TeamMemberRole.Viewer,
  },
])

const addNewInvitee = () => {
  newInvites.value.push({
    key: "",
    value: TeamMemberRole.Viewer,
  })
}

const updateNewInviteeRole = (index: number, role: TeamMemberRole) => {
  newInvites.value[index].value = role
}

const removeNewInvitee = (id: number) => {
  newInvites.value.splice(id, 1)
}

type SendInvitesErrorType =
  | {
      email: Email
      status: "error"
      error: GQLError<CreateTeamInvitationErrors>
    }
  | {
      email: Email
      status: "success"
    }

const sendInvitesResult = ref<Array<SendInvitesErrorType>>([])

const sendingInvites = ref<boolean>(false)

const sendInvites = async () => {
  const validationResult = pipe(
    newInvites.value,
    O.fromPredicate(
      (invites): invites is Array<{ key: Email; value: TeamMemberRole }> =>
        pipe(
          invites,
          A.every((invitee) => EmailCodec.is(invitee.key))
        )
    ),
    O.map(
      A.map((invitee) =>
        createTeamInvitation(invitee.key, invitee.value, props.editingTeamID)
      )
    )
  )

  if (O.isNone(validationResult)) {
    // Error handling for no validation
    toast.error(`${t("error.incorrect_email")}`)
    return
  }

  sendingInvites.value = true

  sendInvitesResult.value = await pipe(
    A.sequence(T.task)(validationResult.value),
    T.chain(
      flow(
        A.mapWithIndex((i, el) =>
          pipe(
            el,
            E.foldW(
              (err) => ({
                status: "error" as const,
                email: newInvites.value[i].key as Email,
                error: err,
              }),
              () => ({
                status: "success" as const,
                email: newInvites.value[i].key as Email,
              })
            )
          )
        ),
        T.of
      )
    )
  )()

  sendingInvites.value = false
}

const getErrorMessage = (error: SendInvitesErrorType) => {
  if (error.type === "network_error") {
    return t("error.network_error")
  } else {
    switch (error.error) {
      case "team/invalid_id":
        return t("team.invalid_id")
      case "team/member_not_found":
        return t("team.member_not_found")
      case "team_invite/already_member":
        return t("team.already_member")
      case "team_invite/member_has_invite":
        return t("team.member_has_invite")
    }
  }
}

const hideModal = () => {
  sendingInvites.value = false
  sendInvitesResult.value = []
  newInvites.value = [
    {
      key: "",
      value: TeamMemberRole.Viewer,
    },
  ]
  emit("hide-modal")
}
</script>
