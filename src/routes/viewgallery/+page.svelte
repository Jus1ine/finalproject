<script lang="ts">
    import { goto } from '$app/navigation';
    import { uploadedImages, type GalleryImage, type Comment } from '$lib';
    import { userProfile } from '$lib';
    import { onMount } from 'svelte';
    
    let isProfileOpen = false;
    let selectedImage: GalleryImage | null = null;
    let newCommentText = '';
    let comments: Comment[] = [];

    function toggleProfile() {
        isProfileOpen = !isProfileOpen;
    }

    function handleProfileKeydown(event: KeyboardEvent) {
        if (event.key === 'Enter' || event.key === ' ') {
            toggleProfile();
        }
    }

    function openImageModal(image: GalleryImage) {
        selectedImage = image;
        comments = image.comments || [];
    }

    function closeImageModal() {
        selectedImage = null;
        comments = [];
        newCommentText = '';
    }

    function addComment() {
        if (!selectedImage || !newCommentText.trim()) return;

        const newComment: Comment = {
            id: Date.now(),
            text: newCommentText.trim(),
            userName: $userProfile.userName,
            timestamp: new Date().toISOString()
        };

        uploadedImages.update(images => 
            images.map(img => 
                img.id === selectedImage?.id 
                    ? { ...img, comments: [...(img.comments || []), newComment] } 
                    : img
            )
        );

        comments = [...comments, newComment];
        newCommentText = '';
    }

    // Function to toggle like on an image
    function toggleLike(imageId: number, event: MouseEvent) {
        event.stopPropagation(); // Prevent opening modal when clicking like button
        uploadedImages.update(images =>
            images.map(img =>
                img.id === imageId
                    ? { 
                        ...img, 
                        likes: img.likes ? 
                            img.likes.includes($userProfile.userName) ?
                                img.likes.filter(user => user !== $userProfile.userName) :
                                [...img.likes, $userProfile.userName] :
                            [$userProfile.userName]
                    }
                    : img
            )
        );
    }

    // Function to share image
    function shareImage(image: GalleryImage, event: MouseEvent) {
        event.stopPropagation(); // Prevent opening modal when clicking share button
        if (navigator.share) {
            navigator.share({
                title: image.name || 'Shared Image',
                text: 'Check out this image from my gallery!',
                url: image.url
            }).catch(console.error);
        } else {
            // Fallback: Copy URL to clipboard
            navigator.clipboard.writeText(image.url)
                .then(() => alert('Image URL copied to clipboard!'))
                .catch(console.error);
        }
    }

    // Function to generate a consistent shape and aspect ratio for images
    function getImageShape(image: GalleryImage): { 
        containerClass: string, 
        imageClass: string 
    } {
        const seed = image.url.length + (image.name?.length || 0) + image.id;
        
        // Define different shape classes with Pinterest-like variations
        const shapeOptions = [
            // Square
            { 
                containerClass: 'aspect-square', // 1:1
                imageClass: 'object-cover' 
            },
            // Tall rectangle
            { 
                containerClass: 'aspect-[2/3]', // Taller rectangle
                imageClass: 'object-cover' 
            },
            // Extra tall rectangle
            { 
                containerClass: 'aspect-[3/5]', // Even taller rectangle
                imageClass: 'object-cover' 
            },
            // Wide rectangle
            { 
                containerClass: 'aspect-[3/2]', // Wider rectangle
                imageClass: 'object-cover' 
            },
            // Extra wide rectangle
            { 
                containerClass: 'aspect-[2/1]', // Even wider rectangle
                imageClass: 'object-cover' 
            },
            // Standard photo
            { 
                containerClass: 'aspect-[4/3]', // Standard photo ratio
                imageClass: 'object-cover' 
            }
        ];
        
        // Use seed to consistently select a shape
        return shapeOptions[seed % shapeOptions.length];
    }
</script>

<div class="min-h-screen bg-white">
    <div class="container mx-auto p-4 relative">
        <div class="flex justify-between items-center mb-6">
            <div class="flex items-center gap-4">
                <h1 class="text-3xl font-bold">View Gallery</h1>
            </div>
            <div class="relative">
                <button 
                    on:click={toggleProfile}
                    class="flex items-center space-x-2 px-4 py-2 text-gray-800 rounded-lg hover:bg-gray-50 focus:outline-none"
                >
                    <img 
                        src={$userProfile.profilePicture} 
                        alt="Profile" 
                        class="w-8 h-8 rounded-full"
                    />
                    <span class="font-medium">{$userProfile.userName}</span>
                </button>

                {#if isProfileOpen}
                    <div class="absolute right-0 mt-2 w-64 bg-white rounded-2xl shadow-lg py-2 z-50 border">
                        <div class="px-4 py-3 border-b">
                            <div class="flex items-center space-x-3">
                                <img 
                                    src={$userProfile.profilePicture} 
                                    alt="Profile" 
                                    class="w-10 h-10 rounded-full object-cover"
                                />
                                <div>
                                    <h3 class="text-sm font-semibold">{$userProfile.userName}</h3>
                                    <p class="text-xs text-gray-500">
                                        {$userProfile.moods.join(', ')}
                                    </p>
                                </div>
                            </div>
                        </div>
                        <div class="py-2">
                            <button
                                on:click={() => goto('/Profile')}
                                class="w-full text-left px-4 py-2 hover:bg-gray-50 flex items-center space-x-3"
                            >
                                <svg class="w-5 h-5 text-gray-500" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                                    <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M16 7a4 4 0 11-8 0 4 4 0 018 8" />
                                </svg>
                                <span>Profile</span>
                            </button>

                            <button
                                on:click={() => goto('/viewgallery')}
                                class="w-full text-left px-4 py-2 hover:bg-gray-50 flex items-center space-x-3"
                            >
                                <svg class="w-5 h-5 text-gray-500" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                                    <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M4 16l4.586-4.586a2 2 0 012.828 0L16 16m-2-2l1.586-1.586a2 2 0 012.828 0L20 14m-6-6h.01M6 20h12a2 2 0 002-2V6a2 2 0 00-2-2H6a2 2 0 00-2 2v12a2 2 0 002 2z" />
                                </svg>
                                <span>View Gallery</span>
                            </button>

                            <button
                                on:click={() => goto('/Gallery')}
                                class="w-full text-left px-4 py-2 hover:bg-gray-50 flex items-center space-x-3"
                            >
                                <svg class="w-5 h-5 text-gray-500" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                                    <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M4 16l4.586-4.586a2 2 0 012.828 0L16 16m-2-2l1.586-1.586a2 2 0 012.828 0L20 14m-6-6h.01M6 20h12a2 2 0 002-2V6a2 2 0 00-2-2H6a2 2 0 00-2 2v12a2 2 0 002 2z" />
                                </svg>
                                <span>Upload Gallery</span>
                            </button>

                            <div class="border-t my-2"></div>

                            <button
                                on:click={() => goto('/settings')}
                                class="w-full text-left px-4 py-2 hover:bg-gray-50 flex items-center space-x-3"
                            >
                                <svg class="w-5 h-5 text-gray-500" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                                    <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M10.325 4.317c.426-1.756 2.924-1.756 3.35 0a1.724 1.724 0 002.573 1.066c1.543-.94 3.31.826 2.37 2.37a1.724 1.724 0 001.065 2.572c1.756.426 1.756 2.924 0 3.35a1.724 1.724 0 00-1.066 2.573c.94 1.543-.826 3.31-2.37 2.37a1.724 1.724 0 00-2.572 1.065c-.426 1.756-2.924 1.756-3.35 0a1.724 1.724 0 00-2.573-1.066c-1.543.94-3.31-.826-2.37-2.37a1.724 1.724 0 00-1.065-2.572c-1.756-.426-1.756-2.924 0-3.35a1.724 1.724 0 001.066-2.573c-.94-1.543.826-3.31 2.37-2.37.996.608 2.296.07 2.572-1.065z" />
                                </svg>
                                <span>Settings</span>
                            </button>

                            <button
                                on:click={() => {
                                    // Add sign out logic here
                                    goto('/');
                                }}
                                class="w-full text-left px-4 py-2 hover:bg-gray-50 flex items-center space-x-3 text-red-500"
                            >
                                <svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                                    <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M17 16l4-4m0 0l-4-4m4 4H7m6 4v1a3 3 0 01-3 3H6a3 3 0 01-3-3V7a3 3 0 013-3h4a3 3 0 013 3v1" />
                                </svg>
                                <span>Sign Out</span>
                            </button>
                        </div>
                    </div>
                {/if}
            </div>
        </div>

        <!-- Gallery Grid -->
        {#if $uploadedImages.length === 0}
            <div class="text-center text-gray-500 py-8">
                <p class="text-xl">No images in the gallery yet</p>
                <p class="mt-2">Upload some images to get started!</p>
            </div>
        {:else}
            <div class="masonry-grid">
                {#each $uploadedImages as image}
                    <div class="masonry-item mb-4 break-inside-avoid" on:click={() => openImageModal(image)}>
                        <div class="relative group rounded-lg overflow-hidden bg-gray-100">
                            {#if image.url}
                                {@const shape = getImageShape(image)}
                                <div class={`relative ${shape.containerClass}`}>
                                    <img
                                        src={image.url}
                                        alt={image.name || 'Gallery image'}
                                        class={`w-full h-full ${shape.imageClass}`}
                                    />
                                    <!-- Heart and Share buttons -->
                                    <div class="absolute top-2 right-2 flex gap-2 opacity-0 group-hover:opacity-100 transition-opacity">
                                        <button
                                            on:click={(e) => toggleLike(image.id, e)}
                                            class="p-2 rounded-full bg-white/90 hover:bg-white shadow-lg transition-transform hover:scale-110"
                                            title={image.likes?.includes($userProfile.userName) ? 'Unlike' : 'Like'}
                                        >
                                            <svg
                                                class={`w-5 h-5 ${image.likes?.includes($userProfile.userName) ? 'text-red-500 fill-current' : 'text-gray-600'}`}
                                                xmlns="http://www.w3.org/2000/svg"
                                                viewBox="0 0 24 24"
                                                stroke="currentColor"
                                                stroke-width="2"
                                            >
                                                <path
                                                    d="M12 21.35l-1.45-1.32C5.4 15.36 2 12.28 2 8.5 2 5.42 4.42 3 7.5 3c1.74 0 3.41.81 4.5 2.09C13.09 3.81 14.76 3 16.5 3 19.58 3 22 5.42 22 8.5c0 3.78-3.4 6.86-8.55 11.54L12 21.35z"
                                                />
                                            </svg>
                                        </button>
                                        <button
                                            on:click={(e) => shareImage(image, e)}
                                            class="p-2 rounded-full bg-white/90 hover:bg-white shadow-lg transition-transform hover:scale-110"
                                            title="Share"
                                        >
                                            <svg
                                                class="w-5 h-5 text-gray-600"
                                                xmlns="http://www.w3.org/2000/svg"
                                                viewBox="0 0 24 24"
                                                fill="none"
                                                stroke="currentColor"
                                                stroke-width="2"
                                            >
                                                <path d="M4 12v8a2 2 0 0 0 2 2h12a2 2 0 0 0 2-2v-8"/>
                                                <polyline points="16 6 12 2 8 6"/>
                                                <line x1="12" y1="2" x2="12" y2="15"/>
                                            </svg>
                                        </button>
                                    </div>
                                    <!-- Like count -->
                                    {#if image.likes?.length}
                                        <div class="absolute bottom-2 right-2 px-2 py-1 rounded-full bg-white/90 text-sm font-medium text-gray-700">
                                            {image.likes.length} {image.likes.length === 1 ? 'like' : 'likes'}
                                        </div>
                                    {/if}
                                </div>
                            {/if}
                        </div>
                    </div>
                {/each}
            </div>
        {/if}
    </div>
</div>

<!-- Image Modal -->
{#if selectedImage}
    <div 
        class="fixed inset-0 bg-black/80 z-50 flex items-center justify-center p-4 overflow-hidden"
        on:click|self={closeImageModal}
    >
        <div class="bg-white rounded-2xl max-w-6xl w-full max-h-[90vh] flex shadow-2xl overflow-hidden">
            <!-- Image Section -->
            <div class="w-2/3 bg-gray-100 flex items-center justify-center p-4">
                <img 
                    src={selectedImage?.url || ''} 
                    alt={selectedImage?.name || 'Gallery Image'} 
                    class="max-w-full max-h-[80vh] object-contain rounded-xl"
                />
            </div>
            
            <!-- Comments Section -->
            <div class="w-1/3 bg-white p-6 overflow-y-auto flex flex-col">
                <!-- Image Details -->
                <div class="mb-6 border-b pb-4">
                    <h2 class="text-xl font-bold mb-2">{selectedImage?.name || 'Untitled Image'}</h2>
                    <p class="text-gray-500 text-sm">
                        Uploaded on {selectedImage ? new Date(selectedImage.timestamp).toLocaleDateString() : ''}
                    </p>
                </div>

                <!-- Comments Header -->
                <div class="flex justify-between items-center mb-4">
                    <h3 class="text-lg font-semibold">
                        Comments 
                        <span class="text-gray-500 text-sm ml-2">({comments.length})</span>
                    </h3>
                </div>
                
                <!-- Comments List -->
                <div class="flex-1 space-y-4 mb-6 overflow-y-auto pr-2">
                    {#if comments.length > 0}
                        {#each comments as comment}
                            <div class="flex space-x-3">
                                <img 
                                    src={$userProfile.profilePicture} 
                                    alt="Profile" 
                                    class="w-8 h-8 rounded-full object-cover flex-shrink-0"
                                />
                                <div>
                                    <div class="bg-gray-100 rounded-xl p-3">
                                        <p class="text-sm font-semibold mb-1">{comment.userName}</p>
                                        <p class="text-sm">{comment.text}</p>
                                    </div>
                                    <p class="text-xs text-gray-500 mt-1">
                                        {new Date(comment.timestamp).toLocaleString('en-US', {
                                            month: 'short', 
                                            day: 'numeric', 
                                            hour: '2-digit', 
                                            minute: '2-digit'
                                        })}
                                    </p>
                                </div>
                            </div>
                        {/each}
                    {:else}
                        <div class="text-center text-gray-500 py-8">
                            <p class="text-2xl mb-2">🤔</p>
                            <p>No comments yet</p>
                        </div>
                    {/if}
                </div>
                
                <!-- Add Comment -->
                <div class="mt-auto border-t pt-4">
                    <div class="flex items-center space-x-3 mb-3">
                        <img 
                            src={$userProfile.profilePicture} 
                            alt="Profile" 
                            class="w-8 h-8 rounded-full object-cover"
                        />
                        <p class="font-medium">{$userProfile.userName}</p>
                    </div>
                    <textarea 
                        bind:value={newCommentText}
                        placeholder="Add a comment..."
                        class="w-full p-3 border border-gray-300 rounded-xl mb-3 resize-none focus:ring-2 focus:ring-blue-500 focus:outline-none"
                        rows="3"
                    ></textarea>
                    <button 
                        on:click={addComment}
                        disabled={!newCommentText.trim()}
                        class="w-full bg-red-500 text-white py-3 rounded-xl hover:bg-red-600 transition-colors disabled:opacity-50 disabled:cursor-not-allowed"
                    >
                        Post Comment
                    </button>
                </div>
            </div>
        </div>
    </div>
{/if}

<style>
    .masonry-grid {
        column-count: 1;
        column-gap: 1rem;
    }

    .masonry-item {
        break-inside: avoid;
        margin-bottom: 1rem;
    }

    @media (min-width: 640px) {
        .masonry-grid {
            column-count: 2;
        }
    }

    @media (min-width: 1024px) {
        .masonry-grid {
            column-count: 3;
        }
    }

    @media (min-width: 1280px) {
        .masonry-grid {
            column-count: 4;
        }
    }
</style>