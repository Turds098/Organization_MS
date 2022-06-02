<template>
  <div class="q-pa-md">
    <q-page>
      <div class="row q-pa-sm q-col-gutter-sm">
        <div class="col-3">
          <q-select
            required
            v-model="form.region_id"
            :options="dataxFilter"
            label="Select Region"
            dense
            bg-color="green-2"
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
        <div class="col-3">
          <q-select
            v-model="form.club_id"
            :options="dataxFilter1"
            label="Select Club"
            dense
            bg-color="green-2"
            map-options
            emit-value
            option-label="c_name"
            option-value="C_id"
            :stack-label="stacklabel"
            filled
            behavior="menu"
          />
        </div>
      </div>

      <q-table
        @row-click="showMenu"
        class="my-sticky-header-table"
        title="Member List"
        :rows="datax"
        :columns="columns"
        row-key="name"
        flat
        bordered
        dense
        :pagination="initialPagination"
        :filter="search"
      >
        <template v-slot:top-right>
          <q-input
            dense
            rounded
            outlined
            debounce="600"
            style="width: 300px"
            v-model="search"
            placeholder="Search"
          >
            <template v-slot:append>
              <q-icon name="search" />
            </template>
          </q-input>
        </template>
      </q-table>

      <q-page-sticky position="bottom-right" :offset="[18, 18]">
        <q-menux
          :showfilter="false"
          :showPrint="false"
          @add="onEntry"
        ></q-menux>
      </q-page-sticky>
    </q-page>
    <entry ref="entryx" :vmodel="showEntry" :datax="form" @close="getList" />
  </div>
</template>
<script>
import { ref, onMounted, vModelSelect, watch } from "vue";
import { api } from "src/boot/axios";
import { onSave, castArray, beforeCancel, initialPaginationx } from "src/utils";
import entry from "./entry";
import { QBtnToggle, useQuasar } from "quasar";
import MyMixin from "src/mixin";
export default {
  props: {
    stacklabel: { type: Boolean, default: true },
    modelValue: String,
    vmodel: Boolean,
  },
  components: { entry },
  setup(props) {
    const datax = ref([]);
    const datax1 = ref([]);
    const dataxFilter = ref([]);
    const dataxFilter1 = ref([]);
    const $q = useQuasar();
    const paginationx = ref(initialPaginationx());
    const form = ref({
      M_id: 0,
      region_id: "",
      m_regionname: "",
      club_id: "",
      m_fullname: "",
      m_email: "",
      m_status: "",
      m_doc: "",
      m_address: "",
    });
    function showMenu(evt, row, index) {
      $q.bottomSheet({
        message: row.subject_name,
        actions: [
          {},
          {
            label: "Update",
            icon: "update",
            id: "Update",
            color: "primary",
          },
          {
            label: "Delete",
            icon: "delete_outline",
            id: "cancel",
            color: "negative",
          },
        ],
      }).onOk((action) => {
        if (action.id === "Update") {
          onUpdate(row);
        }
        if (action.id === "cancel") {
          onCancel(row.id);
        }
      });
    }
    function onCancel(id) {
      beforeCancel().then(() => {
        api.put("Setup/Member/cancel/" + id).then(({ data }) => {
          getList();
        });
      });
    }
    function getClub() {
      api
        .get("Setup/Club/showPerRegion/" + form.value.region_id)
        .then(({ data }) => {
          dataxFilter1.value = data;
        });
    }
    function getRegion() {
      api.get("Setup/Region/show").then(({ data }) => {
        dataxFilter.value = data;
      });
    }
    // function getClub() {
    //   api.get("Setup/Club/showPerRegion"+form.value.region_id).then(({ data }) => {
    //     dataxFilter1.value = data;

    //   });
    // }
    onMounted(async () => {
      await getRegion();

      // await dateNow();
      // await generateRef(form.value.m_doc);
    });
    watch(
      () => form.value.region_id,
      (newData) => {
        // dataxFilter.value = [];
        form.value.club_id = "";
        if (newData > 0) {
          getClub();
          getList();
        }
      }
    );
    watch(
      () => form.value.club_id,
      (newData) => {
        // dataxFilter.value = [];

        if (newData > 0) {
          getList();
        }
      }
    );
    function getList() {
      showEntry.value = false;
      datax.value = [];
      const par = {
        reg: form.value.region_id,
        club: form.value.club_id,
      };
      console.log(par);
      api.post("Setup/Member/show", par).then(({ data }) => {
        datax.value = data;
      });
    }
    const showEntry = ref(false);
    const dlgEntry = ref(false);
    function onEntry() {
      showEntry.value = true;
    }
    async function onUpdate(itm) {
      showEntry.value = true;
      let form = {
        M_id: itm.M_id,
        region_id: itm.region_id,
        club_id: itm.club_id,
        m_fullname: itm.m_fullname,
        m_status: itm.m_status,
        m_email: itm.m_email,
        m_address: itm.m_address,
      };
      entryx.value.form = castArray(form);
      await entryx.value.getClassYear();
    }
    onMounted(async () => {
      await getList();
    });
    const entryx = ref();
    const search = ref("");
    return {
      initialPagination: paginationx,
      onUpdate,
      showMenu,
      dataxFilter1,
      dataxFilter,
      form,
      entryx,
      search,
      dlgEntry,
      showEntry,
      onEntry,
      getList,
      datax1,
      columns: [
        { label: "Full Name", field: "m_fullname", align: "left" },
        { label: "Contact", field: "m_contact", align: "left" },
        { label: "Email", field: "m_email", align: "left" },
        { label: "Member Since", field: "m_doc", align: "left" },
      ],
      datax,
      onItemClick() {
        if (this.onItemClick == "Active") {
          m_status = "Active";
        } else {
          m_status = "InActive";
        }
        console.log("Clicked on an Item");
      },
    };
  },
};
</script>
<style lang="sass">
.my-sticky-header-table


  .q-table__top,
  .q-table__bottom,
  thead tr:first-child th

    background-color: #c1f4cd

  thead tr th
    position: sticky
    z-index: 1
  thead tr:first-child th
    top: 0

  &.q-table--loading thead tr:last-child th
</style>
