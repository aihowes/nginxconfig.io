<!--
Copyright 2020 DigitalOcean

This code is licensed under the MIT License.
You may obtain a copy of the License at
https://github.com/digitalocean/nginxconfig.io/blob/master/LICENSE or https://mit-license.org/

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and / or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions :

The above copyright notice and this permission notice shall be included in
all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT.IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN
THE SOFTWARE.
-->

<template>
    <div class="panel">
        <div class="tabs">
            <ul>
                <li v-for="tab in tabs" :class="tabClass(tab.key)">
                    <a @click="active = tab.key">{{ tab.display }}{{ changes(tab.key) }}</a>
                </li>
            </ul>
        </div>

        <component :is="tab"
                   v-for="tab in tabs"
                   :key="tab.key"
                   :data="$props.data[tab.key]"
                   :style="{ display: active === tab.key ? undefined : 'none' }"
                   class="container"
        ></component>

        <div class="navigation-buttons">
            <a v-if="previousTab !== false" class="button is-mini" @click="active = previousTab">
                <i class="fas fa-long-arrow-alt-left"></i> <span>{{ i18n.common.back }}</span>
            </a>
            <a v-if="nextTab !== false" class="button is-primary is-mini" @click="active = nextTab">
                <span>{{ i18n.common.next }}</span> <i class="fas fa-long-arrow-alt-right"></i>
            </a>
        </div>
    </div>
</template>

<script>
    import i18n from '../i18n';
    import isChanged from '../util/is_changed';
    import * as Sections from './global_sections';

    const tabs = Object.values(Sections);
    const delegated = tabs.reduce((prev, tab) => {
        prev[tab.key] = tab.delegated;
        return prev;
    }, {});

    export default {
        name: 'Global',
        delegated,          // Data the parent will present here
        props: {
            data: Object,   // Data delegated back to us from parent
        },
        data() {
            return {
                i18n,
                active: tabs[0].key,
                tabs,
            };
        },
        computed: {
            nextTab() {
                const tabs = this.$data.tabs.map(t => t.key);
                const index = tabs.indexOf(this.$data.active) + 1;
                if (index < tabs.length) return tabs[index];
                return false;
            },
            previousTab() {
                const tabs = this.$data.tabs.map(t => t.key);
                const index = tabs.indexOf(this.$data.active) - 1;
                if (index >= 0) return tabs[index];
                return false;
            },
        },
        methods: {
            changesCount(tab) {
                return Object.keys(this.$props.data[tab])
                    .filter(key => isChanged(this.$props.data[tab][key], tab, key)).length;
            },
            changes(tab) {
                const changes = this.changesCount(tab);
                if (changes) return ` (${changes.toLocaleString()})`;
                return '';
            },
            setValue(tab, key, val) {
                Object.assign(this.$props.data[tab][key], { value: val, computed: val });
            },
            resetValue(tab, key) {
                this.setValue(tab, key, this.$props.data[tab][key].default);
            },
            tabClass(tab) {
                const classes = [];
                if (tab === this.$data.active) classes.push('is-active');
                if (this.changesCount(tab)) classes.push('is-changed');
                const tabs = this.$data.tabs.map(t => t.key);
                if (tabs.indexOf(tab) < tabs.indexOf(this.$data.active)) classes.push('is-before');
                return classes.join(' ');
            },
        },
    };
</script>
