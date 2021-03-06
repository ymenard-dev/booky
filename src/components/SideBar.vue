<template>
  <div class="side-bar-wrapper">
    <header>
      <button :class="{ active: stashMode }" @click="mode = 'stash'">
        Stash
      </button>
      <button :class="{ active: tabsMode }" @click="mode = 'tabs'">Tabs</button>
    </header>
    <section>
      <smooth-dnd-container
        group-name="tabs"
        class="side-bar-container"
        :should-accept-drop="shouldDndDrop"
        :get-child-payload="getCardPayloadFromTabsList()"
        :drop-placeholder="placeholderOptions"
        :behaviour="dndBehavior"
        @drop="onCardDrop"
      >
        <smooth-dnd-draggable
          v-for="item in items"
          :key="`item-${item.id}`"
          class="item"
        >
          <item
            :item="item"
            :locked="false"
            :text-editable="stashMode && !locked"
            :link-clickable="locked"
            :display-delete-btn="!locked"
            @change="$emit('change')"
            @delete-item="
              (item) => (stashMode ? deleteItem(item) : closeTab(item))
            "
          />
        </smooth-dnd-draggable>
      </smooth-dnd-container>
    </section>
  </div>
</template>

<script>
import {
  Container as SmoothDndContainer,
  Draggable as SmoothDndDraggable,
} from "@ymenard-dev/vue-smooth-dnd";
import Item from "./Item";
import { applyDrag, uuidv4 } from "../utils/utils";

export default {
  name: "SideBar",
  components: {
    SmoothDndContainer,
    SmoothDndDraggable,
    Item,
  },
  props: {
    tabs: {
      type: Array,
      required: true,
    },
    stash: {
      type: Array,
      required: true,
    },
    locked: {
      type: Boolean,
      default: false,
    },
  },
  data() {
    return {
      mode: "stash",
      placeholderOptions: {
        className: "cards-drop-preview",
        animationDuration: "150",
        showOnTop: true,
      },
    };
  },
  computed: {
    tabsMode() {
      return this.mode === "tabs";
    },
    stashMode() {
      return this.mode === "stash";
    },
    items() {
      return this.mode === "tabs" ? this.tabs : this.stash;
    },
    dndBehavior() {
      return this.mode === "tabs" ? "move" : "move";
    },
    hasLink() {
      return this.stashMode && this.locked;
    },
    link() {
      return this.hasLink ? "a" : "div";
    },
  },
  methods: {
    shouldDndDrop(item) {
      return this.mode === "stash";
    },
    getCardPayloadFromTabsList() {
      return (index) => {
        return {
          item: {
            ...this.items[index],
            id: uuidv4(),
          },
          from: this.mode,
        };
      };
    },
    onCardDrop(dropResult) {
      if (this.mode !== "stash") {
        return;
      }

      const stash = applyDrag(this.items, dropResult);
      this.updateStash(stash);
    },
    closeTab(item) {
      const idx = this.items.indexOf(item);
      if (idx >= 0) {
        this.items.splice(idx, 1);
        chrome.tabs.remove(item.tabId);
      }
    },
    deleteItem(item) {
      if (this.mode !== "stash") {
        return;
      }
      const idx = this.items.indexOf(item);
      if (idx >= 0) {
        this.items.splice(idx, 1);
        this.updateStash(this.items);
      }
    },
    updateStash(stash) {
      this.$emit("update:stash", stash);
      this.$emit("change");
    },
  },
};
</script>

<style lang="scss" scoped>
@import "../styles/mixins.scss";

button {
  @include backgroundColor(--primary-color2, 10%, -20%);
  @include fontColor(--font-color-white);
  text-transform: uppercase;
  letter-spacing: 0.25rem;
  border-radius: 5px;
  width: 40%;
  line-height: 25px;

  &hover {
    @include backgroundColor(--primary-color2, 0%, -30%);
  }

  &.active {
    @include backgroundColor(--primary-color2, -20%, 30%);
  }
}

header {
  display: flex;
  justify-content: space-evenly;
  padding: 10px 0;
}

.side-bar-wrapper {
  @include backgroundColor(--primary-color1, -3%, 10%);
  padding: 0 6px;
  flex: 1;
}

.side-bar-container {
  overflow: auto;
  padding: 0 3px;
}

a {
  text-decoration: none;
}

.item {
  margin-bottom: 4px;

  &:first-child {
    padding-top: 0;
  }

  &:last-child {
    padding-bottom: 0;
  }
}
</style>
