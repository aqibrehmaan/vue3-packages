<template>
    <editor v-model="editorValue" :init="init"></editor>
</template>
  
  <script setup>
  import { reactive, ref, watch, toRefs } from 'vue';
  
  import tinymce from 'tinymce/tinymce.js';
  import 'tinymce/skins/ui/oxide/skin.css';
  import 'tinymce/themes/silver';
  
  // Icon
  import 'tinymce/icons/default';
  import 'tinymce/plugins/emoticons';
  import 'tinymce/plugins/emoticons/js/emojis.js';
  import 'tinymce/plugins/table';
  import 'tinymce/plugins/quickbars';
  import 'tinymce/plugins/image';
  import 'tinymce/plugins/media';
  import 'tinymce/plugins/link';
  import 'tinymce/plugins/code';

  // TinyMCE-Vue
  import Editor from '@tinymce/tinymce-vue';
  
  const props = defineProps({
    modelValue: {
      type: String,
      default: '',
    },
    plugins: {
      type: [String, Array],
      default: 'quickbars emoticons table code lists link image',
    },
    toolbar: {
      type: [String, Array],
      default:
        ' bold italic underline strikethrough | fontsizeselect | forecolor backcolor | alignleft aligncenter alignright alignjustify|bullist numlist |outdent indent blockquote | undo redo | axupimgs | removeformat | table | emoticons | link image | code',
    },
  });
  
  const emit = defineEmits(['update:modelValue']);
  
  const init = reactive({
    height: 500,
    menubar: false,
    content_css: false,
    skin: false,
    plugins: props.plugins,
    toolbar: props.toolbar,
    quickbars_insert_toolbar: false,
    branding: false,
    image_title: true,
  /* enable automatic uploads of images represented by blob or data URIs*/
  automatic_uploads: true,
  /*
    URL of our upload handler (for more details check: https://www.tiny.cloud/docs/configure/file-image-upload/#images_upload_url)
    images_upload_url: 'postAcceptor.php',
    here we add custom filepicker only to Image dialog
  */
  file_picker_types: 'image',
  /* and here's our custom image picker*/
  file_picker_callback: function (cb, value, meta) {
    var input = document.createElement('input');
    input.setAttribute('type', 'file');
    input.setAttribute('accept', 'image/*');

    /*
      Note: In modern browsers input[type="file"] is functional without
      even adding it to the DOM, but that might not be the case in some older
      or quirky browsers like IE, so you might want to add it to the DOM
      just in case, and visually hide it. And do not forget do remove it
      once you do not need it anymore.
    */

    input.onchange = function () {
      var file = this.files[0];

      var reader = new FileReader();
      reader.onload = function () {
        /*
          Note: Now we need to register the blob in TinyMCEs image blob
          registry. In the next release this part hopefully won't be
          necessary, as we are looking to handle it internally.
        */
        var id = 'blobid' + (new Date()).getTime();
        var blobCache =  tinymce.activeEditor.editorUpload.blobCache;
        var base64 = reader.result.split(',')[1];
        var blobInfo = blobCache.create(id, file, base64);
        blobCache.add(blobInfo);

        /* call the callback and populate the Title field with the file name */
        cb(blobInfo.blobUri(), { title: file.name });
      };
      reader.readAsDataURL(file);
    };

    input.click();
  },
  });
  
  const { modelValue } = toRefs(props);
  const editorValue = ref(modelValue.value);
  
  watch(modelValue, (newValue) => {
    editorValue.value = newValue;
  });
  
  watch(editorValue, (newValue) => {
    emit('update:modelValue', newValue);
  });
  </script>
  