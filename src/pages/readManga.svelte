<script>
    import Layout from "../components/Layout.svelte";
    import { onMount } from "svelte";
    import Loading from "../components/Loading.svelte";
    import {
        IcnArrowLeft,
        IcnArrowRight,
        IcnClose,
    } from "../components/Icons.svelte";
    import { replace } from "svelte-spa-router";
    import ChapterItem from "../components/ChapterItem.svelte";

    export let params;
    let dataset;
    let loading = true;
    let isChapterBarOpen = false;

    onMount(async () => {
        call(params.chapter);
    });

    async function call(chapter, rplc = true) {
        loading = true;
        if (rplc) {
            replace(`/manga/${params.slug}/${chapter}`);
        }

        await fetch(`/api/manga/${params.slug}/${chapter}`)
            .then((r) => r.json())
            .then((data) => {
                dataset = data;
                loading = false;
            });
    }

    var prevScrollpos = window.pageYOffset;
    window.onscroll = function () {
        var currentScrollPos = window.pageYOffset;
        var nav = document.getElementById("navbar");
        if (nav) {
            if (prevScrollpos > currentScrollPos) {
                nav.classList.add("bottom-4");
                nav.classList.remove("-bottom-20");
            } else {
                nav.classList.add("-bottom-20");
                nav.classList.remove("bottom-4");
            }
            prevScrollpos = currentScrollPos;
        }
    };
</script>

{#if !loading}
    <div
        class={`w-full h-full fixed top-0 bg-black ${
            isChapterBarOpen ? "opacity-50" : "opacity-0 invisible"
        } z-20 transition-all duration-300 ease-in-out`}
    />
    <Layout px="0" showNav={false}>
        <h1 class="text-3xl text-center px-3 font-bold">
            {dataset.title}
        </h1>
        <div class="m-auto w-full">
            {#each dataset.data as url, i}
                <img
                    class="w-full"
                    src={url}
                    alt={`${dataset.title}-image-${i + 1}`}
                />
            {/each}
        </div>

        <nav
            class={`fixed w-full xl:max-w-5xl ${
                isChapterBarOpen ? "bottom-0" : "-bottom-full"
            } bg-gray-800 z-30 rounded-tr-xl rounded-tl-xl transition-all duration-300 ease-in-out`}
        >
            <div class="p-3 space-y-3">
                <h2 class="text-center font-bold text-lg">Pilih Chapter</h2>
                <div class="overflow-y-auto max-h-96">
                    <div class="grid grid-cols-5 xl:grid-cols-12 gap-3">
                        {#each dataset.chapters as chapter}
                            <div on:click={call(chapter)}>
                                <ChapterItem
                                    isSelected={chapter === params.chapter}
                                    {chapter}
                                    slug={params.slug}>{chapter}</ChapterItem
                                >
                            </div>
                        {/each}
                    </div>
                </div>
                <button
                    class="p-1 fill-current w-full flex items-center justify-center"
                    on:click={() => (isChapterBarOpen = !isChapterBarOpen)}
                    ><IcnClose /></button
                >
            </div>
        </nav>

        <nav
            id="navbar"
            class="fixed left-0 flex justify-center items-center bottom-4 w-full z-10 transition-all duration-300 ease-in-out"
        >
            <div
                class="flex justify-between w-1/2 xl:w-1/6 p-3 rounded-full bg-gray-800 shadow-md"
            >
                {#if dataset.prev}
                    <button
                        class="p-1 fill-current rounded-full"
                        on:click={call(dataset.prev)}
                    >
                        <IcnArrowLeft />
                    </button>
                {/if}
                <button
                    on:click={() => (isChapterBarOpen = !isChapterBarOpen)}
                    class="p-1 fill-current rounded-full"
                >
                    {params.chapter}
                </button>
                {#if dataset.next}
                    <button
                        on:click={call(dataset.next)}
                        class="p-1 fill-current rounded-full"
                    >
                        <IcnArrowRight />
                    </button>
                {/if}
            </div>
        </nav>
    </Layout>
{:else}
    <Loading />
{/if}
