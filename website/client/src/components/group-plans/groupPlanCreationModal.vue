<template>
  <b-modal
    id="create-group"
    :title="$t('createGroupTitle')"
    :hide-footer="true"
    :hide-header="true"
    size="md"
    @hide="onHide()"
  >
    <div class="col-12">
      <!-- HEADER -->
      <div
        class="modal-close"
      >
        <span
          class="cancel-text"
          @click="close()"
        >
          {{ $t('cancel') }}
        </span>
        <button
          class="btn btn-primary next-button"
          :value="$t('create')"
          :disabled="!newGroupIsReady"
          @click="createGroup(newGroup)"
        >
          {{ $t('create') }}
        </button>
      </div>
      <h2>{{ $t('createGroup') }}</h2>

      <!-- FORM -->
      <div class="form-group">
        <lockable-label
          :text="$t('nameStar')"
        />
        <input
          id="new-group-name"
          v-model="newGroup.name"
          class="form-control input-medium option-content name-input"
          required="required"
          type="text"
          :placeholder="$t('nameStarText')"
        >
      </div>
      <div class="form-group">
        <lockable-label
          for="new-group-description"
          :text="$t('descriptionOptional')"
          class="description-label"
        />
        <div class="characters-remaining">
          {{ $t('charactersRemaining', {characters: charactersRemaining}) }}
        </div>
        <textarea
          id="new-group-description"
          v-model="newGroup.description"
          class="form-control option-content description-input"
          cols="3"
          :placeholder="$t('descriptionOptionalText')"
          maxlength="250"
        ></textarea>
      </div>
      <div class="form-group">
        <div class="custom-control custom-checkbox">
          <input
            id="create-group-leaderOnlyChallenges-checkbox"
            v-model="newGroup.leaderOnly.challenges"
            class="custom-control-input"
            type="checkbox"
          >
          <label
            class="custom-control-label"
            for="create-group-leaderOnlyChallenges-checkbox"
          >{{ $t('leaderOnlyChallenges') }}</label>
        </div>
      </div>
      <div class="form-group">
        <button
          class="btn btn-primary btn-lg btn-block btn-payment"
          :disabled="!newGroupIsReady"
          @click="createGroup(newGroup)"
        >
          {{ $t('create') }}
        </button>
      </div>
    </div>
  </b-modal>
</template>

<style lang="scss" scoped>
  @import '~@/assets/scss/colors.scss';

  h2 {
    color: $purple-300;
    font-size: 1.25rem;
    height: 28px;
    width: 120px;
    margin-top: 24px;
    align-self: center;
  }

  .cancel-text {
    color: $blue-10;
    font-size: 0.875rem;
    margin-right: 16px;
    text-align: center;
    cursor: pointer;
  }
  .next-button {
    align-self: center;
  }

  .form-control::placeholder {
    color: $gray-50;
    height: 32px;
  }

  .description-label {
    margin-bottom: -24px;
  }

  .name-input, .description-input, .group-input {
    margin-top: -4px;
  }

 .characters-remaining {
  color: $gray-100;
  font-size: 0.75rem;
  line-height: 1.33;
  text-align: right;
  margin-bottom: 12px;
 }

  .description-input {
    height: 56px;
  }

  .spacer {
    margin-bottom: 16px !important;
  }

  .btn-payment {
    margin: 24px 112px 24px 112px;
    width: 177px;
  }

  .payments {
    padding: 24px;
  }

  .payment-options {
    margin-bottom: 4em;

    .purple-box {
      background-color: #4f2a93;
      color: #fff;
      padding: .5em;
      border-radius: 8px;
      width: 200px;
      height: 215px;

      .number {
        font-size: 60px;
      }

      .name {
        width: 100px;
        margin-left: .3em;
      }

      div {
        display: inline-block;
      }
    }

    .box, .purple-box {
      display: inline-block;
      vertical-align: bottom;
    }
  }

</style>
<style lang="scss">
  @import '~@/assets/scss/mixins.scss';
  #create-group {
    .modal-dialog {
      max-width: 448px;
    }
    .modal-content {
      width: 448px;
      max-height: 436px;
      border-radius: 8px;
      box-shadow: 0 14px 28px 0 rgba(26, 24, 29, 0.24), 0 10px 10px 0 rgba(26, 24, 29, 0.28);
    }
    .modal-body{
      padding: 0px;
      margin-left: 12px;
      margin-right: 12px;
    }
    .modal-close {
      position: absolute;
      right: 16px;
      cursor: pointer;
      top: 0px;
    }
  }
</style>

<script>
import axios from 'axios';
import pick from 'lodash/pick';
import { CONSTANTS, setLocalSetting } from '@/libs/userlocalManager';
import paymentsMixin from '../../mixins/payments';
import { mapState } from '@/libs/store';
import lockableLabel from '@/components/tasks/modal-controls/lockableLabel';

export default {
  components: {
    lockableLabel,
  },
  mixins: [paymentsMixin],
  data () {
    return {
      newGroup: {
        type: 'guild',
        privacy: 'private',
        name: '',
        description: '',
        leaderOnly: {
          challenges: false,
        },
        demographics: null,
        user: '',
      },
      type: 'guild',
    };
  },
  computed: {
    ...mapState({ user: 'user.data' }),
    newGroupIsReady () {
      return Boolean(this.newGroup.name);
    },
    charactersRemaining () {
      const currentLength = this.newGroup.description ? this.newGroup.description.length : 0;
      return 250 - currentLength;
    },
  },
  methods: {
    close () {
      this.$root.$emit('bv::hide::modal', 'create-group');
    },
    onHide () {
      this.sendingInProgress = false;
    },
    async createGroup (group) {
      this.$root.$emit('bv::hide::modal', 'create-group');

      try {
        const response = await axios.post('/api/v4/groups/create-plan', {
          groupToCreate: group,
          paymentType: '',
        });

        const newGroup = response.data.data;
        if (newGroup && newGroup._id) {
          this.user.guilds.push(newGroup._id);

          const appState = {
            newGroup: true,
            paymentCompleted: true,
            group: pick(newGroup, ['_id', 'memberCount', 'name']),
          };
          setLocalSetting(CONSTANTS.savedAppStateValues.SAVED_APP_STATE, JSON.stringify(appState));

          this.$router.push(`/group-plans/${newGroup._id}/task-information`);
          // const habiticaUrl = `${window.location.protocol}//${window.location.host}`;
          // window.location.assign(`${habiticaUrl}/group-plans/${newGroup._id}/task-information`);
          return;
        }

        window.location.reload(true);
      } catch (e) {
        console.log(e); // eslint-disable-line no-console
      }
    },
  },
};
</script>
