<template>
    <div id="top-horizontal-scroll" class="table-wrapper">
        <table v-if="data.length !== 0">
            <thead>
            <tr>
                <th class="table-th-actions"
                    title="Quick actions">
                    <font-awesome-icon icon="tools"/>
                </th>
                <th class="table-th"
                    :id="struct.Field"
                    :class="struct.Key === 'PRI' ? 'text-indigo-800' : 'text-gray-800'"
                    :title="struct.Field + ' - ' + struct.Type"
                    :type="struct.Type"
                    @click="$emit('columnSelect', $event)"
                    v-for="struct in structure">
                    {{readability ? enhanceReadability(struct.Field) : struct.Field}}
                    <br>
                    <p class="text-xs font-light text-gray-700 -mt-1" @click.prevent>
                        {{struct.Type}}</p>
                </th>
            </tr>
            </thead>
            <tbody>
            <tr v-if="data" class="table-row" v-for="row in data">
                <td class="table-td-actions" title="Inspect row">
                    <font-awesome-icon style="transform: rotate(90deg);" icon="search-plus"/>&nbsp;
                </td>
                <td class="ellipsis table-td"
                    :id="item ? item : ENUM.PREQUEL_UNDEFINED"
                    :class="!item ? 'text-gray-500 italic' : 'text-gray-700'"
                    :title="(item ? item + ` (Length ${(item + '').length})` : 'This item is empty') + '\nLeft click to see\nRight click to edit'"
                    :contenteditable="false"
                    @contextmenu.prevent="dataModifier($event)"
                    @click="seeCompleteData($event)"
                    @focusout="resetFocus($event)"
                    v-for="item in row">
                    {{item ? item : 'Nothing here'}}
                </td>
            </tr>
            </tbody>
        </table>
        <TableEmpty v-if="data.length === 0" :structure="structure"/>
    </div>
</template>

<script>
  import TableEmpty from './TableEmpty';

  export default {
    name      : 'Table',
    components: {TableEmpty},
    props     : ['structure', 'data', 'readability'],

    data() {
      return {

        /**
         * Holds data about table view
         */
        view: {
          params: new URLSearchParams(window.location.search),
          cell  : {
            selected : {},
            editing  : false,
            saveEvent: {},
          },
        },

        /**
         * Enumerator
         */
        ENUM: {
          PREQUEL_UNDEFINED: 'PREQUEL_UNDEFINED',
        },
      };
    },

    methods: {

      seeCompleteData: function(ev) {
        if (ev.target.classList.contains('ellipsis') && ev.target.contentEditable === 'false') {
          ev.target.classList.remove('ellipsis');
        }
        else if (ev.target.contentEditable === 'false') {
          ev.target.classList.add('ellipsis');
        }
      },

      resetFocus: function(ev) {
        ev.target.classList.remove('bg-white', 'border', 'cursor-text');
        ev.target.classList.add('ellipsis', 'hover:bg-gray-300');
        ev.target.contentEditable = false;
        this.view.cell.selected   = {};
      },

      /**
       * Save data @focusout
       *
       * @TODO
       */
      saveModifiedData: function() {
        this.view.cell.selected.classList.remove('bg-white', 'border', 'cursor-text');
        this.view.cell.selected.classList.add('ellipsis', 'hover:bg-gray-300');
        this.view.cell.selected.contentEditable = false;
        this.view.cell.selected                 = {};
      },

      /**
       * At right click, open contextmenu to edit data.
       * @param ev $event
       */
      dataModifier: function(ev) {
        this.view.cell.selected = ev.target;
        this.view.cell.editing  = true;

        this.view.cell.selected.contentEditable = true;
        this.view.cell.selected.classList.remove('ellipsis', 'hover:underline', 'hover:bg-gray-300');
        this.view.cell.selected.classList.add('bg-white', 'border', 'cursor-text');
        this.view.cell.selected.focus();
      },

      /**
       * Enhance readability for column names.
       *
       * @param str
       * @returns {string}
       */
      enhanceReadability: function(str) {
        if (!this.$props.readability) {
          return str;
        }

        let words    = str.split(/[!@#$%^&*(),.?":{}|<>_-]/);
        let readable = '';

        for (let i = 0; i < words.length; i++) {
          readable += capitalise(words[i].toLowerCase());

          if (i !== (words.length - 1)) {
            readable += ' ';
          }
        }

        return readable;
      },
    },
  };
</script>

<style lang="scss">
    #top-horizontal-scroll {
        transform: rotateX(180deg);
    }

    .ellipsis {
        width: 250px;
        max-width: 250px;
        white-space: nowrap;
        overflow: hidden;
        text-overflow: ellipsis;
        transition: all .5s ease;
    }

    .table-wrapper {
        table {
            @apply w-full;
            @apply rounded;
            @apply bg-gray-200;

            thead {
                @apply border-b;
                @apply rounded;
                @apply bg-gray-400;

                .table-th {
                    @apply border;
                    @apply p-1;
                    @apply whitespace-no-wrap;
                    @apply text-sm;
                    @apply mx-10;
                    @apply text-center;
                    @apply cursor-pointer;

                    &:hover {
                        @apply bg-gray-300;
                    }
                }


                .table-th-actions {
                    @apply border;
                    @apply p-2;
                    @apply text-sm;
                    @apply text-gray-800;
                    @apply text-center;
                    @apply cursor-pointer;

                    &:hover {
                        @apply bg-gray-300;
                        @apply border;
                    }
                }

            }

            .table-row:nth-child(odd) {
                @apply bg-gray-100;
            }

            .table-row:nth-child(even) {
                @apply bg-gray-200;
            }

            .table-td {
                @apply px-4;
                @apply text-sm;
                @apply text-center;
                @apply cursor-pointer;

                &:hover {
                    @apply bg-gray-300;
                }
            }

            .table-td-actions {
                @apply text-gray-700;
                @apply w-8;
                @apply text-sm;
                @apply text-center;
                @apply cursor-pointer;

                &:hover {
                    @apply bg-gray-400;
                }
            }
            transform: rotateX(180deg);
        }

    }
</style>
