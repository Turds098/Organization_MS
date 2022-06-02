<template>
  <q-dialog v-model="show" full-height position="right">
    <q-card style="width: 500px; max-width: 100vw">
      <q-toolbar class="bg-primary text-white">
        <q-btn @click="onBack" flat round dense icon="arrow_back" />
        <q-toolbar-title>Member Setup</q-toolbar-title>
      </q-toolbar>
      <q-separator />

    <div class=" q-pb-md column items-center" >
       <div class=" q-px-xl q-pt-sm q-my-none ">
            <q-uploader
              rounded
              bordered
              color="green-10"
              hide-upload-btn
              style="width: 100%"
              label="Upload Region Logo"
              multiple
              @added="uploadfile"
              @removed="fileRemove"
            />
      </div>
        </div>

      <div class="row q-pa-sm q-col-gutter-sm">
        <div class="col-6">
          <q-select
                required v-model="form.region_id"
                :options="dataxFilter"
                label="Select Region"
                dense
                map-options
                emit-value
                option-label="R_Name"
                option-value="R_id"
                :stack-label="stacklabel"
                filled
                use-input
                behavior="menu"
                input-debounce="0"

          />
        </div>
              <div class="col-6">
              <q-select
              v-model="form.club_id"
              :options="dataxFilter1"
              label="Select Club"
              dense
              map-options
              emit-value
              option-label="c_name"
              option-value="C_id"
              :stack-label="stacklabel"
              filled
              use-input
              behavior="menu"
              input-debounce="0"
                    />
                  </div>
         <div class="col-12">
          <q-input
            v-model="form.m_fullname"
            type="text"
            label="Fullname"
            stack-label
            filled
            dense
          />
        </div>
          <div class="col-12">
          <q-input
            v-model="form.m_contact"
            type="text"
            label="Contact "
            stack-label
            filled
            dense
          />
        </div>

           <div class="col-12">
          <q-input
            v-model="form.m_email"
            type="text"
            label="Email "
            stack-label
            filled
            dense
          />
        </div>

        <div class="col-12" id="status">
            <q-btn-toggle
      v-model="form.m_status"
      push
      glossy
      toggle-color="primary"
      :options="[
        {label: 'Active', value: 'Active'},
        {label: 'Inactive', value: 'Inactive'}
      ]"
    />
        </div>
        <div class="col-12">
          <q-command
            @save="onSave"
            @clear="onClear"
            :labelSave="form.M_id > 0 ? 'Update' : 'Save' "
          />
        </div>
      </div>
    </q-card>
  </q-dialog>
</template>
<script>
import { useStore } from "vuex";
import { ref, onMounted, watch, toRefs } from "vue";
import { api } from "src/boot/axios";
import {
  beforeSave,
  success,
  castArray,
  showLoading,
  hideLoading,
} from "src/utils";
export default {
  data() {
      return {
        previewImage: null
      };
    },
  methods: {
      selectImage () {
          this.$refs.fileInput.click()
      },
      pickFile () {
        let input = this.$refs.fileInput
        let file = input.files
        if (file && file[0]) {
          let reader = new FileReader
          reader.onload = e => {
            this.previewImage = e.target.result
          }
          reader.readAsDataURL(file[0])
          this.$emit('input', file[0])
        }
      }
  },
  name: "Member",
  props: {
    stacklabel: { type: Boolean, default: true },
    modelValue: String,
    vmodel: Boolean,
  },

  setup(props, { emit }) {
    const show = toRefs(props).vmodel;
    const options = ref();
    const form = ref({
      M_id: 0,
      m_fullname: "",
      region_id: "",
      club_id: "",
      m_contact: "",
      m_email:"",
      m_status:"",
      m_doc:"",
    });
    const uploadData = ref([]);
    function uploadfile(file){
      file.forEach((element) => uploadData.value.push(element));
    }
    function fileRemove(file){
      var filtered = uploadData.value.filter(function (value, index,arr){
        return value.name !== file[0].name;
      });
      uploadData.value = filtered;
    }
    const classList = ref([]);
     const datax = ref([]);
     const datax1 = ref([]);
    const dataxFilter = ref([]);
     const dataxFilter1 = ref([]);

    // function getclassType() {
    //   api.get("Setup/ClassType/getClassType").then(({ data }) => {
    //     classList.value = data;
    //   });
    // }
    function onBack() {
      emit("close");
      onClear();
    }
    //  const yearList = ref([]);
    // function getClassYear() {
    //   api
    //     .get("Setup/ClassType/getClassYear/" + form.value.class_type_id)
    //     .then(({ data }) => {
    //       yearList.value = data;
    //     });
    // }

    function getRegion() {
      //alert("64545");
      api.get("Setup/Region/show").then(({ data }) => {
        dataxFilter.value = data;
      });
    }
    function getClub() {

      api.get("Setup/Club/showPerRegion/"+form.value.region_id).then(({ data }) => {

        dataxFilter1.value = data;
      });
    }
    function Uploadfiles(id){
      let files = uploadData.value;
      const data = new FormData();
      for (var i = 0; i < files.length; i++){
        let file = files [i];
        data.append(`files[${i}]`,file);
      }
      data.append("id", id);

    }

    function onSave() {
      if((form.value.region_id=="")||(form.value.c_regioncode=="")||(form.value.c_name==""))
      {
        alert("Fill all Fields")
      }
      else
      {


      beforeSave(form.value.id < 1).then(() => {
        showLoading();
        let par = { form: form.value };
        api.post("Setup/Member/store", par).then(({ data }) => {
        hideLoading();
        success();
        onBack();
        dateNow();
        });
      });
    }
    }
      function onChange(val) {
      let datx = datax.value.find((v) => v.id === val);
      emit("input", datx);
    }

    // }
    function onClear() {
      form.value = {
      M_id: 0,
      m_fullname: "",
      region_id: "",
      club_id: "",
      m_contact: "",
      m_email:"",

      };
    }
    function dateNow() {
        let current_datetime = new Date();
        form.value. m_doc = current_datetime
        .toUTCString()
        .slice(0, 16);
      }
    onMounted(async () => {
       await getRegion();
      await dateNow();
   //   await generateRef(form.value.m_doc);

    });
    watch(
      () => form.value.region_id,
      (newData) => {
      form.value.club_id = "";
        if (newData > 0) {
          getClub();
        }
      }
    );
    return {
      getClub,
      dataxFilter,
      dataxFilter1,
      datax1,
      datax,
      getRegion,
      onChange,
      onClear,
     // classList,
      onSave,
      onBack,
      fileRemove,
      uploadfile,
      show,
      //yearList,
      form,
      options,
      filterFn(val, update) {
        if (val === "") {
          update(() => {
            dataxFilter.value = datax.value;
          });
          return;
        }
        update(() => {
          const needle = val.toLowerCase();
          dataxFilter.value = datax.value.filter(
            (v) => v.class_class.toLowerCase().indexOf(needle) > -1
          );
        });
      },
       filterFn1(val, update) {
        if (val === "") {
          update(() => {
            dataxFilter1.value = datax1.value;
          });
          return;
        }
        update(() => {
          const needle = val.toLowerCase();
          dataxFilter1.value = datax1.value.filter(
            (v) => v.class_class.toLowerCase().indexOf(needle) > -1
          );
        });
      },
      onItemClick () {
        if(this.onItemClick == 'Active')
        {
            m_status='Active'
        }
        else{
          m_status='InActive'
        }
        console.log('Clicked on an Item')

      }
    };
  },
};

</script>

