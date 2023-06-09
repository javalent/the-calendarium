<script lang="ts">
    import type Calendarium from "src/main";
    import {
        normalizePath,
        TextComponent as ObsidianTextComponent,
        TFolder,
    } from "obsidian";
    import { FolderSuggestionModal } from "src/suggester/folder";
    import { getContext } from "svelte";
    import TextAreaComponent from "../Settings/TextAreaComponent.svelte";
    import TextComponent from "../Settings/TextComponent.svelte";
    import ToggleComponent from "../Settings/ToggleComponent.svelte";
    import Details from "../Utilities/Details.svelte";
    import { DEFAULT_CALENDAR } from "src/settings/settings.constants";

    export let plugin: Calendarium;

    const calendar = getContext("store");

    $: displayDayNumber = $calendar.static.displayDayNumber;
    $: incrementDay = $calendar.static.incrementDay;

    $: validName = $calendar.name != null && $calendar.name.length;

    $: autoParse = $calendar.autoParse;

    $: supportInlineEvents = $calendar.supportInlineEvents;

    if (!$calendar.inlineEventTag)
        $calendar.inlineEventTag = DEFAULT_CALENDAR.inlineEventTag;

    const folder = (node: HTMLElement) => {
        let folders = plugin.app.vault
            .getAllLoadedFiles()
            .filter((f) => f instanceof TFolder);
        const text = new ObsidianTextComponent(node);
        if (!$calendar.path) $calendar.path = "/";
        text.setPlaceholder($calendar.path ?? "/");
        const modal = new FolderSuggestionModal(plugin.app, text, [
            ...(folders as TFolder[]),
        ]);

        modal.onClose = async () => {
            const v = text.inputEl.value?.trim()
                ? text.inputEl.value.trim()
                : "/";
            $calendar.path = normalizePath(v);
        };

        text.inputEl.onblur = async () => {
            const v = text.inputEl.value?.trim()
                ? text.inputEl.value.trim()
                : "/";
            $calendar.path = normalizePath(v);
        };
    };

    $: inlineEventTagDesc = createFragment((e) => {
        e.createSpan({
            text: "Tag to specify which notes to scan for inline events, e.g. ",
        });
        e.createEl("code", { text: "inline-events" });
        e.createSpan({
            text: " to use the ",
        });
        e.createEl("code", { text: "#inline-events" });
        e.createSpan({
            text: " tag.",
        });
    });

    const inlineEventTagSetting = (node: HTMLElement) => {
        const text = new ObsidianTextComponent(node);
        text.setValue(
            `${$calendar.inlineEventTag ?? ""}`.replace("#", "")
        ).onChange(async (v) => {
            $calendar.inlineEventTag = v.startsWith("#") ? v : `#${v}`;
            await plugin.saveSettings();
        });
    };
</script>

<Details
    name={"Basic Info"}
    warn={!validName}
    label={"The calendar must have a name"}
>
    <div class="calendarium-info">
        <TextComponent
            name={"Calendar Name"}
            warn={!validName}
            desc={!validName ? "The calendar must have a name" : ""}
            value={$calendar.name}
            on:blur={(evt) => {
                $calendar.name = evt.detail;
            }}
            on:change={(evt) => ($calendar.name = evt.detail)}
        />
        <TextAreaComponent
            name={"Calendar Description"}
            value={$calendar.description}
            on:blur={(evt) => ($calendar.description = evt.detail)}
        />
        <ToggleComponent
            name={"Display Day Number"}
            desc={"Display day of year in Day View"}
            value={displayDayNumber}
            on:click={() => {
                $calendar.static.displayDayNumber =
                    !$calendar.static.displayDayNumber;
            }}
        />
        <ToggleComponent
            name={"Auto Increment Day"}
            desc={"Automatically increment the current day every real-world day."}
            value={incrementDay}
            on:click={() => {
                $calendar.static.incrementDay = !$calendar.static.incrementDay;
            }}
        />
        <ToggleComponent
            name={"Parse Files for Events"}
            desc={"The plugin will automatically parse files in the vault for events."}
            value={autoParse}
            on:click={() => {
                $calendar.autoParse = !$calendar.autoParse;
            }}
        />
        {#if autoParse}
            <TextComponent
                name={"Events Folder"}
                desc={"The plugin will only parse files in this folder for events."}
                value={$calendar.path}
            >
                <div use:folder />
            </TextComponent>
            <ToggleComponent
                name={"Support Inline Events"}
                desc={"Look for <span> tags defining events in notes."}
                value={supportInlineEvents}
                on:click={() => {
                    $calendar.supportInlineEvents =
                        !$calendar.supportInlineEvents;
                }}
            />
            {#if supportInlineEvents}
                {#key $calendar.supportInlineEvents}
                    <TextComponent
                        name={"Default Inline Events Tag"}
                        desc={inlineEventTagDesc}
                        value={""}
                    >
                        <div
                            use:inlineEventTagSetting
                            class="setting-item-control"
                        />
                    </TextComponent>
                {/key}
            {/if}
        {/if}
    </div>
</Details>

<style>
    .calendarium-info :global(.setting-item) {
        padding-top: 18px;
    }
    .calendarium-info :global(.calendarium-description) {
        display: flex;
        flex-flow: column;
        align-items: flex-start;
    }
    .calendarium-info :global(.calendarium-description) :global(textarea) {
        width: 100%;
    }
</style>
