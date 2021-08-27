<template>
  <div>
    <validate-discard-action
      :dataset="dataset"
      @discard-records="onDiscard"
      @validate-records="onValidate"
    ></validate-discard-action>
    <create-new-action @new-label="onNewLabel"></create-new-action>
  </div>
</template>

<script>
import { mapActions } from "vuex";
import "assets/icons/export";
import "assets/icons/plus";
import "assets/icons/refresh";

export default {
  props: {
    dataset: {
      type: Object,
      required: true,
    },
  },
  data: () => ({}),

  methods: {
    ...mapActions({
      discard: "entities/datasets/discardAnnotations",
      validate: "entities/datasets/validateAnnotations",
    }),

    async onDiscard(records) {
      await this.discard({
        dataset: this.dataset,
        records: records,
      });
    },
    async onValidate(records) {
      await this.validate({
        dataset: this.dataset,
        records: records.map((record) => ({
          ...record,
          annotation: {
            ...(record.annotation || record.prediction),
            agent: this.$auth.user,
          },
        })),
      });
    },
    async onNewLabel(label) {
      await this.dataset.$dispatch("setEntities", {
        dataset: this.dataset,
        entities: [
          ...new Set([...this.dataset.entities.map((ent) => ent.text), label]),
        ],
      });
    },
  },
};
</script>
