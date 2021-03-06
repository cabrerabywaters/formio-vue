<template>
    <div>
        <div ref="formio" class="formContainer">
        </div>
    </div>
</template>
<script>
import { Formio as FormioFull } from "formiojs/full";

export default {
  name: "formio",
  props: {
    src: {
      type: String
    },
    form: {
      type: Object
    },
    submission: {
      type: Object
    },
    options: {
      type: Object,
      default: () => ({})
    }
  },
  mounted() {
    this.initializeForm()
      .then(() => {
        this.setupForm();
      })
      .catch(err => {
        /* eslint-disable no-console */
        console.warn(err);
        /* eslint-enable no-console */
      });
  },

  destroyed() {
    if (this.formio) {
      this.formio.destroy(true);
    }
  },

  methods: {
    initializeForm() {
      return new Promise((resolve, reject) => {
        if (this.src) {
          resolve(
            FormioFull.createForm(
              this.$refs.formio,
              this.src,
              this.options
            ).then(formio => {
              this.formio = formio;
              return formio;
            })
          ).catch(err => {
            /* eslint-disable no-console */
            console.error(err);
            /* eslint-enable no-console */
          });
        } else if (this.form) {
          resolve(
            FormioFull.createForm(
              this.$refs.formio,
              this.form,
              this.options
            ).then(formio => {
              this.formio = formio;
              return formio;
            })
          );
        } else {
          // If we get to here there is no src or form
          reject(new Error("Must set src or form attribute"));
        }
      });
    },

    setupForm() {
      if (this.submission) {
        this.formio.submission = this.submission;
      }

      if (this.url) {
        this.formio.url = this.url;
      }

      this.formio.events.onAny((...args) => {
        const eventParts = args[0].split(".");

        // Only handle formio events.
        if (eventParts[0] !== "formio" || eventParts.length !== 2) {
          return;
        }

        // Remove formio. from event.
        args[0] = eventParts[1];

        this.$emit.apply(this, args);

        // Emit custom events under their own name as well.
        if (eventParts[1] === "customEvent") {
          args[0] = args[1].type;
          this.$emit.apply(this, args);
        }
      });
    }
  },

  watch: {
    src: function(value) {
      if (this.formio) {
        this.formio.src = value;
      }
    },
    url: function(value) {
      if (this.formio) {
        this.formio.url = value;
      }
    },
    form: function(value) {
      if (this.formio) {
        this.formio.form = value;
      }
    },
    submission: function(value) {
      if (this.formio) {
        this.formio.submission = value;
      }
    }
  }
};
</script>
