<script lang="ts">
    import { onMount } from 'svelte';

    interface GalleryImage {
        id: number;
        url: string;
        name: string;
        timestamp: string;
    }

    let images: GalleryImage[] = [];

    onMount(() => {
        const storedImages = localStorage.getItem('galleryImages');
        if (storedImages) {
            images = JSON.parse(storedImages) as GalleryImage[];
        }
    });
</script>

<div class="container mx-auto p-4">
    <h1 class="text-3xl font-bold mb-6">Your Images</h1>
    
    {#if images.length === 0}
        <div class="text-center text-gray-500 py-8">
            <p>No images uploaded yet. Go to the Gallery to upload some images!</p>
        </div>
    {:else}
        <div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-4">
            {#each images as image (image.id)}
                <div class="relative">
                    <img 
                        src={image.url} 
                        alt={image.name}
                        class="w-full h-64 object-cover rounded-lg shadow-md"
                    />
                    <div class="mt-2">
                        <p class="text-sm text-gray-600">{image.name}</p>
                        <p class="text-xs text-gray-400">{new Date(image.timestamp).toLocaleDateString()}</p>
                    </div>
                </div>
            {/each}
        </div>
    {/if}
</div>