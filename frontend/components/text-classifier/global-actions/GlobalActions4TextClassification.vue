<template>
  <div>
    <validate-discard-action
      :dataset="dataset"
      @discard-records="onDiscard"
      @validate-records="onValidate"
    >
      <template slot-scope="props">
        <feedback-dropdown-all
          class="global-actions__select"
          :multi-label="isMultiLabelRecord"
          :options="options"
          @selected="onSelectAnnotation($event, props.selectedRecords)"
        ></feedback-dropdown-all>
      </template>
    </validate-discard-action>

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
  computed: {
    isMultiLabelRecord() {
      return this.dataset.visibleRecords.some((record) => record.multi_label);
    },
    options() {
      const record = this.dataset.results.records[0];
      let labels =
        record && record.prediction
          ? record.prediction.labels.map((label) => label.class)
          : [];
      labels = Array.from(new Set([...labels, ...this.dataset.labels]));
      return labels;
    },
  },
  methods: {
    ...mapActions({
      updateRecords: "entities/datasets/updateRecords",
      discard: "entities/datasets/discardAnnotations",
      validate: "entities/datasets/validateAnnotations",
    }),
    async onSelectAnnotation(labels, selectedRecords) {
      const records = selectedRecords.map((record) => {
        const appliedLabels = record.annotation
          ? [...record.annotation.labels]
          : [];

        const filterAppliedLabels = labels.filter(
          (l) => appliedLabels.map((label) => label.class).indexOf(l) === -1
        );

        let newLabels = this.isMultiLabelRecord ? filterAppliedLabels : labels;
        newLabels = newLabels.map((label) => ({
          class: label,
          confidence: 1.0,
        }));
        return {
          ...record,
          annotation: {
            agent: this.$auth.user,
            labels: this.isMultiLabelRecord
              ? [...appliedLabels, ...newLabels]
              : newLabels,
          },
        };
      });
      await this.validate({ dataset: this.dataset, records: records });
    },
    async onDiscard(records) {
      await this.discard({
        dataset: this.dataset,
        records: records,
      });
    },
    async onValidate(records) {
      await this.validate({
        dataset: this.dataset,
        records: records,
      });
    },
    async onNewLabel(newLabel) {
      if (this.isTextClassification) {
        await this.dataset.$dispatch("setLabels", {
          dataset: this.dataset,
          labels: [...new Set([...this.dataset.labels, newLabel])],
        });
      }
    },
  },
};
</script>
<style lang="css" scoped>
.global-actions__select {
  margin-left: 0.8em;
  ::v-deep .dropdown__header {
    border-radius: 5px;
    max-height: 33px;
    background: $lighter-color;
    border-width: 1px;
    color: $font-secondary;
    font-weight: bold;
  }
}
</style>
