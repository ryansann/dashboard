<script>
import { MANAGEMENT, NORMAN } from '@/config/types';
import ArrayList from '@/components/form/ArrayList';
import Loading from '@/components/Loading';
import { _CREATE, _VIEW } from '@/config/query-params';
import { get, set } from '@/utils/object';

function normalizeId(id) {
  return id?.replace(':', '/') || id;
}

// Matches creator-cluster-owner and creator-project-owner
// const CREATOR_OWNER_REGEX = /creator-(cluster|project)-owner/g;

export default {
  components: { ArrayList, Loading },

  props: {
    addMemberDialogName: {
      type:     String,
      required: true
    },

    parentKey: {
      type:     String,
      required: true
    },

    parentId: {
      type:    String,
      default: null
    },

    mode: {
      type:     String,
      required: true
    },

    type: {
      type:     String,
      required: true
    },

    defaultBindingHandler: {
      type:     Function,
      default:  null,
    },
  },

  async fetch() {
    const userHydration = [
      this.$store.dispatch(`management/findAll`, { type: MANAGEMENT.ROLE_TEMPLATE }),
      this.$store.dispatch('rancher/findAll', { type: NORMAN.PRINCIPAL }),
    ];
    const allBindings = this.schema ? await this.$store.dispatch(`management/findAll`, { type: this.type }) : [];
    const bindings = allBindings
      // .filter(b => !b.isSystem || CREATOR_OWNER_REGEX.test(b.metadata.name || ''))
      .filter(b => normalizeId(get(b, this.parentKey)) === normalizeId(this.parentId));

    this.$set(this, 'lastSavedBindings', [...bindings]);

    // Add the current user as the project owner. This will get created by default
    if (this.mode === _CREATE && bindings.length === 0 && this.defaultBindingHandler) {
      const defaultBinding = await this.defaultBindingHandler();

      defaultBinding.isDefaultBinding = true;
      bindings.push(defaultBinding);
    }

    await Promise.all(userHydration);

    this.$set(this, 'bindings', bindings);
  },

  data() {
    return {
      schema:            this.$store.getters[`management/schemaFor`](this.type),
      bindings:                  [],
      lastSavedBindings:         [],
    };
  },

  computed: {
    hasOwnerBinding() {
      return this.bindings.some(b => b.roleTemplate.id.includes('owner'));
    },
    newBindings() {
      return this.bindings
        .filter(binding => !binding.id && !this.lastSavedBindings.includes(binding) && !binding.isDefaultBinding);
    },
    removedBindings() {
      return this.lastSavedBindings
        .filter(binding => !this.bindings.includes(binding));
    },
    membershipUpdate() {
      const newBindings = this.newBindings;
      const removedBindings = this.removedBindings;

      return {
        newBindings:     this.newBindings,
        removedBindings: this.removedBindings,
        save:            (parentId) => {
          const savedPromises = newBindings.map((binding) => {
            set(binding, this.parentKey, parentId);

            return binding.save();
          });

          const removedPromises = removedBindings.map(binding => binding.remove());

          return Promise.all([...savedPromises, ...removedPromises]);
        }
      };
    },

    isCreate() {
      return this.mode === _CREATE;
    },

    isView() {
      return this.mode === _VIEW;
    },
  },
  watch: {
    hasOwnerBinding() {
      this.$emit('has-owner-changed', this.hasOwnerBinding);
    },

    membershipUpdate: {
      deep: true,
      handler() {
        this.$emit('membership-update', this.membershipUpdate);
      }
    }
  },

  methods: {
    addMember() {
      this.$store.dispatch('cluster/promptModal', { component: this.addMemberDialogName, resources: [this.onAddMember] });
    },

    onAddMember(bindings) {
      this.$set(this, 'bindings', [...this.bindings, ...bindings]);
    },
  }
};
</script>
<template>
  <Loading v-if="$fetchState.pending" />
  <ArrayList
    v-else
    v-model="bindings"
    :mode="mode"
    :show-header="true"
  >
    <template #column-headers>
      <div class="box mb-0">
        <div class="column-headers row">
          <div class="col span-6">
            <label class="text-label">{{ t('membershipEditor.user') }}</label>
          </div>
          <div class="col span-6">
            <label class="text-label">{{ t('membershipEditor.role') }}</label>
          </div>
        </div>
      </div>
    </template>
    <template #columns="{row}">
      <div class="columns row">
        <div class="col span-6">
          <Principal :key="row.value.principalId" :value="row.value.principalId" />
        </div>
        <div class="col span-6 role">
          {{ row.value.roleDisplay }}
        </div>
      </div>
    </template>
    <template #add>
      <button
        type="button"
        class="btn role-primary mt-10"
        @click="addMember"
      >
        {{ t('generic.add') }}
      </button>
    </template>
    <template #remove-button="{remove, i}">
      <span v-if="(isCreate && i === 0) || isView || isOnlyRegisteredUser" />
      <button v-else type="button" :disabled="isView" class="btn role-link" @click="remove">
        {{ t('generic.remove') }}
      </button>
    </template>
  </ArrayList>
</template>

<style lang="scss" scoped>
.role {
  display: flex;
  align-items: center;
  flex-direction: row;
}
</style>
