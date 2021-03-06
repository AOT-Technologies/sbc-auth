<template>
  <div>
    <v-fade-transition>
      <div class="loading-container" v-if="!termsContent">
        <v-progress-circular
          size="50"
          width="5"
          color="primary"
          :indeterminate="termsContent"
        />
      </div>
    </v-fade-transition>
    <div v-html="termsContent" class="terms-container"></div>
  </div>
</template>

<script lang="ts">
import { Component, Emit, Prop, Vue } from 'vue-property-decorator'
import { mapActions, mapMutations, mapState } from 'vuex'
import CommonUtils from '@/util/common-util'
import { TermsOfUseDocument } from '@/models/TermsOfUseDocument'
import { User } from '@/models/user'
import documentService from '@/services/document.services.ts'

@Component({
  computed: {
    ...mapState('user', [
      'termsOfUse',
      'userProfile',
      'userHasToAcceptTOS'
    ])
  },
  methods: {
    ...mapActions('user', [
      'getTermsOfUse'
    ])
  }
})
export default class TermsOfUse extends Vue {
  private readonly getTermsOfUse!: (docType?: string) => TermsOfUseDocument
  private termsContent = ''
  protected readonly userProfile!: User
  protected readonly userHasToAcceptTOS!: boolean

  @Prop({ default: 'termsofuse' }) tosType: string

  async mounted () {
    const termsOfService = await this.getTermsOfUse(this.tosType)
    this.termsContent = termsOfService.content
    const hasLatestTermsAccepted = this.hasAcceptedLatestTos(termsOfService.versionId)
    if (!hasLatestTermsAccepted) {
      this.$emit('tos-version-updated')
    }
  }

  private hasAcceptedLatestTos (latestVersionId: string) {
    /*
    version id comes with a string prefix like d1 , d2... strip that , convert to number for comparison
    Or else 'd1' will be,l 'd2' . But 'd2' wont be less than ' d10 '!!!  '
    */

    const userTOS = this.userProfile?.userTerms?.termsOfUseAcceptedVersion

    if (!userTOS) {
      return true
    }
    const currentlyAcceptedTermsVersion = CommonUtils.extractAndConvertStringToNumber(userTOS)
    const latestVersionNumber = CommonUtils.extractAndConvertStringToNumber(latestVersionId)
    return (currentlyAcceptedTermsVersion > latestVersionNumber)
  }
}
</script>

<style lang="scss">
@import '$assets/scss/theme.scss';

// Terms and Conditions Container
$indent-width: 1rem;

.terms-container {
  p, li {
    margin: 1rem 0;
  }

  ol {
    margin-left: $indent-width;
  }

  li {
    padding-left: $indent-width;
  }

  h3 {
    max-width: 55ch;
  }

  section {
    margin-bottom: 2rem;
  }

  section:last-child {
    margin-bottom: 0;
  }

  section header {
    margin-bottom: 1rem;
    color: $gray9;
    font-size: 1rem;
    font-weight: 700 !important;
  }

  section header > span {
    display: inline-block;
    width: $indent-width;
  }

  section a {
    font-weight: 600;
  }

  header + div {
    margin-left: 3.25rem;
  }

  section div > p {
    padding-left: $indent-width;
  }

  p {
    position: relative;
  }

  p + div {
    margin-left: $indent-width;
  }

  p > span {
    position: absolute;
    top: 0;
    left: 0;
  }
}
</style>
