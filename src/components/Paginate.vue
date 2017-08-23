<template>
    <div class="pagination-wrap" v-if="pageCount > 1">
        <ul :class="containerClass">
            <li :class="[prevClass]" v-if="!firstPageSelected()">
                <a @click="firstPage()" @keyup.enter="firstPage()" :class="prevLinkClass" tabindex="0">
                    <icon name="angle-double-left"></icon>
                </a>
            </li><li :class="[prevClass]" v-if="!firstPageSelected()">
                <a @click="prevPage()" @keyup.enter="prevPage()" :class="prevLinkClass" tabindex="0">
                    <icon name="angle-left"></icon>
                </a>
            </li><li v-for="page in pages" :class="[pageClass, { active: page.selected, disabled: page.disabled }]">
                <a v-if="page.disabled" :class="pageLinkClass" tabindex="0">{{ page.content }}</a>
                <a v-else @click="handlePageSelected(page.index)" @keyup.enter="handlePageSelected(page.index)" :class="pageLinkClass" tabindex="0">{{ page.content }}</a>
            </li><li :class="[nextClass]" v-if="!lastPageSelected()">
                <a @click="nextPage()" @keyup.enter="nextPage()" :class="nextLinkClass" tabindex="0">
                    <icon name="angle-right"></icon>
                </a>
            </li><li :class="[nextClass]" v-if="!lastPageSelected()">
                <a @click="lastPage()" @keyup.enter="lastPage()" :class="nextLinkClass" tabindex="0">
                    <icon name="angle-double-right"></icon>
                </a>
            </li>
        </ul>
    </div>
</template>

<script>
import 'vue-awesome/icons'
import Icon from 'vue-awesome/components/Icon.vue'

export default {
    components: {
        Icon
    },
    props: {
        initialPage: {
            type: Number,
            default: 0
        },
        forcePage: {
            type: Number
        },
        clickHandler: {
            type: Function,
            default: () => {}
        },
        pageRange: {
            type: Number,
            default: 3
        },
        marginPages: {
            type: Number,
            default: 1
        },
        containerClass: {
            type: String
        },
        pageClass: {
            type: String
        },
        pageLinkClass: {
            type: String
        },
        prevClass: {
            type: String
        },
        prevLinkClass: {
            type: String
        },
        nextClass: {
            type: String
        },
        nextLinkClass: {
            type: String
        }
    },
    data() {
        return {
            selected: this.initialPage,
            pageCount: 0,
        }
    },
    beforeUpdate() {
        if (this.forcePage === undefined) return
        if (this.forcePage !== this.selected) {
            this.selected = this.forcePage
        }
    },
    computed: {
        pages: function() {
            let items = {}
            if (this.pageCount <= this.pageRange) {
                for (let index = 0; index < this.pageCount; index++) {
                    let page = {
                        index: index,
                        content: index + 1,
                        selected: index === this.selected
                    }
                    items[index] = page
                }
            } else {
                let leftPart = this.pageRange / 2
                let rightPart = this.pageRange - leftPart

                if (this.selected < leftPart) {
                    leftPart = this.selected
                    rightPart = this.pageRange - leftPart
                } else if (this.selected > this.pageCount - this.pageRange / 2) {
                    rightPart = this.pageCount - this.selected
                    leftPart = this.pageRange - rightPart
                }

                // items logic extracted into this function
                let mapItems = index => {
                    let page = {
                        index: index,
                        content: index + 1,
                        selected: index === this.selected
                    }

                    if (index <= this.marginPages - 1 || index >= this.pageCount - this.marginPages) {
                        items[index] = page
                        return
                    }

                    let breakView = {
                        content: '...',
                        disabled: true
                    }

                    if ((this.selected - leftPart) > this.marginPages && items[this.marginPages] !== breakView) {
                        items[this.marginPages] = breakView
                    }

                    if ((this.selected + rightPart) < (this.pageCount - this.marginPages - 1) && items[this.pageCount - this.marginPages - 1] !== breakView) {
                        items[this.pageCount - this.marginPages - 1] = breakView
                    }

                    let overCount = this.selected + rightPart - this.pageCount + 1

                    if (overCount > 0 && index === this.selected - leftPart - overCount) {
                        items[index] = page
                    }

                    if ((index >= this.selected - leftPart) && (index <= this.selected + rightPart)) {
                        items[index] = page
                        return
                    }
                }

                // 1st - loop thru low end of margin pages
                for (let i = 0; i < this.marginPages; i++) {
                    mapItems(i);
                }

                // 2nd - loop thru high end of margin pages
                for (let i = this.pageCount - 1; i >= this.pageCount - this.marginPages; i--) {
                    mapItems(i);
                }

                // 3rd - loop thru selected range
                let selectedRangeLow = 0;
                if (this.selected - this.pageRange > 0) {
                    selectedRangeLow = this.selected - this.pageRange;
                }

                let selectedRangeHigh = this.pageCount;
                if (this.selected + this.pageRange < this.pageCount) {
                    selectedRangeHigh = this.selected + this.pageRange;
                }

                for (let i = selectedRangeLow; i < selectedRangeHigh; i++) {
                    mapItems(i);
                }

            }
            return items
        }
    },
    methods: {
        handlePageSelected(selected) {
            if (this.selected === selected) return

            this.selected = selected

            this.clickHandler(this.selected + 1)
        },
        prevPage() {
            if (this.selected <= 0) return;
            this.selected--;
            this.clickHandler(this.selected + 1);
        },
        firstPage() {
            this.selected = 0;
            this.clickHandler(this.selected + 1);
        },
        lastPage() {
            this.selected = this.pageCount - 1;
            console.log(this.selected + 1);
            this.clickHandler(this.selected + 1);
        },
        nextPage() {
            if (this.selected >= this.pageCount - 1) return;
            this.selected++;
            this.clickHandler(this.selected + 1);
        },
        firstPageSelected() {
            return this.selected === 0
        },
        lastPageSelected() {
            return (this.selected === this.pageCount - 1) || (this.pageCount === 0)
        }
    }
}
</script>

<style lang="css" scoped>
a {
  cursor: pointer;
}
ul { user-select: none; }
</style>
