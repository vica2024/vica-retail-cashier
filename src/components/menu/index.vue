<script lang="tsx">
import { defineComponent, ref, h, compile, computed } from "vue";
import { useI18n } from "vue-i18n";
import { useRoute, useRouter, RouteRecordRaw } from "vue-router";
// import type { RouteMeta } from 'vue-router';
import { useAppStore } from "@/store";
import { listenerRouteChange } from "@/utils/route-listener";
import { openWindow, regexUrl } from "@/utils";
import useMenuTree from "./use-menu-tree";

export default {
	emit: ["collapse"],
	setup() {
		const { t } = useI18n();
		const appStore = useAppStore();
		const router = useRouter();
		const route = useRoute();
		const { menuTree } = useMenuTree();
		const collapsed = computed({
			get() {
				if (appStore.device === "desktop") return appStore.menuCollapse;
				return false;
			},
			set(value) {
				appStore.updateSettings({ menuCollapse: value });
			},
		});

		const topMenu = computed(() => appStore.topMenu);
		const openKeys = ref([]);
		const selectedKey = ref([]);

		const goto = (item) => {
			// Open external link
			if (regexUrl.test(item.path)) {
				openWindow(item.path);
				selectedKey.value = [item.name];
				return;
			}
			// Eliminate external link side effects
			const { hideInMenu, activeMenu } = item.meta;
			if (route.name === item.name && !hideInMenu && !activeMenu) {
				selectedKey.value = [item.name];
				return;
			}
			// Trigger router change
			router.push({
				name: item.name,
			});
		};
		const findMenuOpenKeys = (target) => {
			const result = [];
			let isFind = false;
			const backtrack = (item, keys) => {
				if (item.name === target) {
					isFind = true;
					result.push(...keys);
					return;
				}
				if (item.children?.length) {
					item.children.forEach((el) => {
						backtrack(el, [...keys, el.name]);
					});
				}
			};
			menuTree.value.forEach((el) => {
				if (isFind) return; // Performance optimization
				backtrack(el, [el.name]);
			});
			return result;
		};
		listenerRouteChange((newRoute) => {
			const { requiresAuth, activeMenu, hideInMenu } = newRoute.meta;
			if (requiresAuth && (!hideInMenu || activeMenu)) {
				const menuOpenKeys = findMenuOpenKeys(activeMenu || newRoute.name);

				const keySet = new Set([...menuOpenKeys, ...openKeys.value]);
				openKeys.value = [...keySet];

				selectedKey.value = [activeMenu || menuOpenKeys[menuOpenKeys.length - 1]];
			}
		}, true);
		const setCollapse = (val) => {
			if (appStore.device === "desktop") appStore.updateSettings({ menuCollapse: val });
		};

		const renderSubMenu = () => {
			function travel(_route, nodes = []) {
				if (_route) {
					_route.forEach((element) => {
						// This is demo, modify nodes as needed
						const icon = element?.meta?.icon ? () => h(compile(`<${element?.meta?.icon}/>`),{style: {
                color: 'white',
                fontSize: '50px',
                marginRight:'-15px'
              }}) : null;
						const node =
							element?.children && element?.children.length !== 0 ? (
								<a-sub-menu style="flex-direction: column;text-align: center; padding-top: 10px;color: #fff;"
									key={element?.name}
									v-slots={{
										icon,
										title: () => h(compile(t(element?.meta?.locale || ""))),
									}}
								>
									{travel(element?.children)}
								</a-sub-menu>
							) : (
								<a-menu-item style="flex-direction: column;text-align: center;padding-top: 10px;color: #fff;" key={element?.name} v-slots={{ icon }} onClick={() => goto(element)}>
									{t(element?.meta?.locale || "")}
								</a-menu-item>
							);
						nodes.push(node);
					});
				}
				return nodes;
			}
			return travel(menuTree.value);
		};

		return () => (
			<a-menu
				mode={topMenu.value ? "horizontal" : "vertical"}
				v-model:collapsed={collapsed.value}
				v-model:open-keys={openKeys.value}
				show-collapse-button={appStore.device !== "mobile"}
				auto-open={false}
				selected-keys={selectedKey.value}
				auto-open-selected={true}
				level-indent={34}
				style="height: 100%;width:100%;"
				onCollapse={setCollapse}
			>
				{renderSubMenu()}
			</a-menu>
		);
	},
};
</script>

<style lang="less" scoped>
:deep(.arco-menu-inner) {
	.arco-menu-inline-header {
		display: flex;
		align-items: center;
	}
	.arco-icon {
		&:not(.arco-icon-down) {
			font-size: 18px;
		}
	}
  .arco-menu-item{
    line-height: 18px !important;
    padding-bottom: 15px;
    font-size: 20px;
  }
}
</style>