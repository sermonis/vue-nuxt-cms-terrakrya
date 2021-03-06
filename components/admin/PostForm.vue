<template>
  <ValidationObserver v-slot="{ validate, invalid }">
    <b-form @submit.prevent="validate().then(save)">
      <b-row>
        <b-col md="12">
          <b-form-group label="Título *">
            <validation-provider v-slot="{ errors }" name="título" rules="required">
              <b-form-input v-model="form.title" name="title" />
              <span class="text-danger">{{ errors[0] }}</span>
            </validation-provider>
          </b-form-group>
        </b-col>
        <b-col md="12">
          <b-form-group label="Introdução">
            <b-form-textarea v-model="form.description" name="description" />
          </b-form-group>
        </b-col>
        <b-col md="12">
          <b-form-group label="Descrição completa">
            <quill-editor ref="quillEdit" v-model="form.content" />
            <input id="quillfile" type="file" hidden @change="quillUpload">
          </b-form-group>
        </b-col>
        <b-col md="12">
          <pictures-upload :form="form" field="picture" url="/api/uploads/images" label="Foto de capa" />
        </b-col>
        <b-col md="12">
          <pdfs-upload :form="form" field="pdfs" url="/api/uploads/pdfs" :multiple="true" />
        </b-col>
        <b-col md="12">
          <tags-form v-model="form.tags" :current-tags="currentTags" />
        </b-col>
      </b-row>
      <b-button class="mb-4 mt-4" type="submit" variant="primary" block :disabled="invalid">
        Salvar
      </b-button>
    </b-form>
  </ValidationObserver>
</template>

<script>
import { ValidationObserver, ValidationProvider } from 'vee-validate'
import mixinGlobal from '@/mixins/global'
import mixinForm from '@/mixins/form'
import PicturesUpload from '@/components/admin/PicturesUpload'
import PdfsUpload from '@/components/admin/PdfsUpload'

export default {
  components: {
    ValidationObserver,
    ValidationProvider,
    PicturesUpload,
    PdfsUpload
  },
  mixins: [mixinGlobal, mixinForm],
  props: {
    post: {
      type: Object,
      default: null
    }
  },
  data () {
    return {
      currentTags: [],
      form: {
        title: '',
        description: '',
        content: '',
        picture: null,
        tags: [],
        pdfs: []
      }
    }
  },
  async created () {
    this.toForm(this.form, this.post)
    this.currentTags = await this.$axios.$get('/api/posts/current_tags').catch(this.showError)
  },
  methods: {
    async save () {
      if (this.post) {
        const post = await this.$axios.$put('/api/posts/' + this.post.slug, this.form).catch(this.showError)
        if (post) {
          this.$toast.success('Notícia atualizada com sucesso!')
          this.$router.push('/admin/posts')
        }
      } else {
        const post = await this.$axios.$post('/api/posts', this.form).catch(this.showError)
        if (post) {
          this.$toast.success('Notícia cadastrada com sucesso!')
          this.$router.push('/admin/posts')
        }
      }
    }
  }
}
</script>
