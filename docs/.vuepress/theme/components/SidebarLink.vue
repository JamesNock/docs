<script>
import { isActive, hashRE, groupHeaders } from "../util";

export default {
  functional: true,

  props: ["item", "sidebarDepth"],

  render(
    h,
    {
      parent: { $page, $site, $route, $themeConfig, $themeLocaleConfig },
      props: { item, sidebarDepth },
    }
  ) {
    // use custom active class matching logic
    // due to edge case of paths ending with / + hash
    const selfActive = isActive($route, item.path);

    // for sidebar: auto pages, a hash link should be active if one of its child
    // matches
    const active =
      item.type === "auto"
        ? selfActive ||
          item.children.some((c) =>
            isActive($route, item.basePath + "#" + c.slug)
          )
        : selfActive;
    const link =
      item.type === "external"
        ? renderExternal(h, item.path, item.title || item.path)
        : renderLink(h, item.path, item.title || item.path, active);

    const maxDepth = [
      sidebarDepth,
      $themeLocaleConfig.sidebarDepth,
      $themeConfig.sidebarDepth,
      1,
    ].find((depth) => depth !== undefined);

    const displayAllHeaders =
      $themeLocaleConfig.displayAllHeaders || $themeConfig.displayAllHeaders;

    if (item.type === "auto") {
      return [
        link,
        renderChildren(h, item.children, item.basePath, $route, maxDepth),
      ];
    } else if (
      (active || displayAllHeaders) &&
      item.headers &&
      !hashRE.test(item.path)
    ) {
      const children = groupHeaders(item.headers);
      return [link, renderChildren(h, children, item.path, $route, maxDepth)];
    } else {
      return link;
    }
  },
};

function renderLink(h, to, text, active, level) {
  const component = {
    props: {
      to,
      activeClass: "",
      exactActiveClass: "",
    },
    class: {
      active,
      "sidebar-link": true,
    },
  };

  if (level > 1) {
    component.style = {
      "padding-left": (level * 0.375) + "rem",
    };
  }

  return h("RouterLink", component, text);
}

function renderChildren(h, children, path, route, maxDepth, depth = 1) {
  if (!children || depth > maxDepth) return null;
  return h(
    "ul",
    { class: "sidebar-sub-headers" },
    children.map((c) => {
      // don’t render children beyond maxDepth
      if (c.level && ((c.level - 1) > maxDepth)) return;
      const active = isActive(route, path + "#" + c.slug);
      return h("li", { class: "sidebar-sub-header" }, [
        renderLink(h, path + "#" + c.slug, c.title, active, c.level - 1),
        renderChildren(h, c.children, path, route, maxDepth, depth + 1),
      ]);
    })
  );
}

function renderExternal(h, to, text) {
  return h(
    "a",
    {
      attrs: {
        href: to,
        target: "_blank",
        rel: "noopener noreferrer",
      },
      class: {
        "sidebar-link": true,
      },
    },
    [text, h("OutboundLink")]
  );
}
</script>

<style lang="postcss">
.sidebar-sub-headers {
  font-size: 0.95em;
}

a.sidebar-link {
  @apply px-2 py-1 font-normal inline-block w-full box-border;
  color: var(--sidebar-link-color);
  line-height: 1.4;
  transition:
    color 0.25s ease-out,
    opacity 0.25s ease-out;

  .sidebar-group & {
    @apply pl-4;
  }

  .sidebar-sub-headers & {
    @apply py-1;
    border-left: none;

    &.active {
      @apply font-medium;
    }
  }

  &:hover,
  &.active {
    color: var(--sidebar-active-link-color);
  }
}
</style>
