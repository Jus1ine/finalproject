<script lang="ts">
    import { goto } from '$app/navigation';
    import { uploadedImages, type GalleryImage, userProfile } from '$lib';
    import { onMount } from 'svelte';

    let showEditModal = false;
    let fileInput: HTMLInputElement;

    // Local state for editing
    let newUserName = '';
    let newProfilePicture = '';
    let selectedMoods: string[] = [];

    const moods = [
        { value: 'Professional', color: 'bg-blue-50 text-blue-600' },
        { value: 'Active', color: 'bg-green-50 text-green-600' },
        { value: 'Chill', color: 'bg-purple-50 text-purple-600' },
        { value: 'Happy', color: 'bg-yellow-50 text-yellow-600' },
        { value: 'Sad', color: 'bg-red-50 text-red-600' },
        { value: 'Angry', color: 'bg-orange-50 text-orange-600' },
        { value: 'Nothing', color: 'bg-gray-50 text-gray-600' }
    ];

    function toggleEditModal() {
        showEditModal = !showEditModal;
        if (showEditModal) {
            newUserName = $userProfile.userName;
            newProfilePicture = $userProfile.profilePicture;
            selectedMoods = $userProfile.moods;
        }
    }

    function saveProfileChanges() {
        userProfile.update(profile => ({
            ...profile,
            userName: newUserName,
            profilePicture: newProfilePicture || profile.profilePicture,
            moods: selectedMoods
        }));
        showEditModal = false;
    }

    function handleFileSelect(event: Event) {
        const input = event.target as HTMLInputElement;
        if (input.files && input.files[0]) {
            const reader = new FileReader();
            reader.onload = (e) => {
                newProfilePicture = e.target?.result as string;
            };
            reader.readAsDataURL(input.files[0]);
        }
    }

    function exitProfile() {
        goto('/viewgallery');
    }
</script>

<div class="min-h-screen bg-[#DAE0E6]">
    <!-- Profile Header -->
    <div class="relative pt-16">
        <!-- Back Button -->
        <button 
            on:click={exitProfile}
            class="fixed top-4 left-4 px-4 py-2 bg-white/90 backdrop-blur-sm rounded-full text-gray-700 hover:bg-white transition-all duration-200 flex items-center space-x-2 shadow-sm z-10"
        >
            <svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M10 19l-7-7m0 0l7-7m-7 7h18" />
            </svg>
            <span>Back</span>
        </button>
        
        <div class="max-w-5xl mx-auto px-4 mt-24">
            <div class="relative -mt-20">
                <div class="bg-white rounded-lg shadow-lg">
                    <!-- Profile Header -->
                    <div class="p-6">
                        <div class="flex flex-col sm:flex-row items-start gap-6">
                            <!-- Avatar -->
                            <div class="w-32 h-32 rounded-full overflow-hidden border-4 border-white shadow-lg">
                                <img 
                                    src={$userProfile.profilePicture} 
                                    alt="Profile" 
                                    class="w-full h-full object-cover"
                                />
                            </div>
                            
                            <div class="flex-1">
                                <div class="flex justify-between items-center">
                                    <div>
                                        <h2 class="text-2xl font-bold text-gray-900">{$userProfile.userName}</h2>
                                        <p class="text-sm text-gray-500">u/{$userProfile.userName} • Joined 2 years ago</p>
                                        <div class="mt-3 flex flex-wrap gap-2">
                                            {#each $userProfile.moods as mood}
                                                {#if moods.find(m => m.value === mood)}
                                                    <span class="px-3 py-1 text-sm {moods.find(m => m.value === mood)?.color ?? 'bg-gray-50 text-gray-600'} rounded-full font-medium">
                                                        {mood}
                                                    </span>
                                                {/if}
                                            {/each}
                                        </div>
                                    </div>
                                    
                                    <button 
                                        on:click={toggleEditModal}
                                        class="px-4 py-2 bg-[#FF4500] text-white rounded-md hover:bg-[#FF5722] transition-colors"
                                    >
                                        Edit Profile
                                    </button>
                                </div>
                            </div>
                        </div>
                    </div>

                    <!-- Navigation Tabs -->
                    <div class="border-t border-gray-200">
                        <nav class="flex space-x-8 px-6" aria-label="Profile navigation">
                            <button class="px-1 py-4 text-sm font-medium text-[#FF4500] border-b-2 border-[#FF4500]">
                                Posts
                            </button>
                            <button class="px-1 py-4 text-sm font-medium text-gray-500 hover:text-gray-700">
                                About
                            </button>
                        </nav>
                    </div>
                </div>
            </div>
        </div>
    </div>

    <!-- Image Grid (Pinterest Style) -->
    <div class="max-w-7xl mx-auto px-4 py-8">
        {#if $uploadedImages.length === 0}
            <div class="text-center text-gray-500 py-8">
                <p>No images uploaded yet. Start uploading in the Gallery!</p>
            </div>
        {:else}
            <div class="columns-1 sm:columns-2 md:columns-3 lg:columns-4 gap-4">
                {#each $uploadedImages as image}
                    <div class="break-inside-avoid mb-4">
                        <div class="bg-white rounded-lg overflow-hidden shadow-sm hover:shadow-xl transition-shadow duration-300">
                            <img 
                                src={image.url} 
                                alt={image.name}
                                class="w-full h-auto object-cover"
                            />
                        </div>
                    </div>
                {/each}
            </div>
        {/if}
    </div>
</div>

<!-- Edit Profile Modal -->
{#if showEditModal}
    <div class="fixed inset-0 bg-black bg-opacity-50 z-50 flex items-center justify-center">
        <div class="bg-white rounded-xl shadow-2xl w-full max-w-md mx-auto transform transition-all duration-300 ease-in-out scale-100 hover:shadow-3xl">
            <div class="p-6">
                <div class="flex justify-between items-center mb-6">
                    <h2 class="text-2xl font-bold text-gray-800">Edit Profile</h2>
                    <button 
                        on:click={() => showEditModal = false}
                        class="text-gray-500 hover:text-gray-800 transition-colors rounded-full p-2 hover:bg-gray-100"
                    >
                        <svg xmlns="http://www.w3.org/2000/svg" class="h-6 w-6" fill="none" viewBox="0 0 24 24">
                            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M6 18L18 6M6 6l12 12" />
                        </svg>
                    </button>
                </div>
                
                <div class="space-y-4">
                    <div>
                        <label class="block text-sm font-medium text-gray-700 mb-2">Profile Picture</label>
                        <div 
                            class="w-32 h-32 mx-auto rounded-lg overflow-hidden cursor-pointer relative group"
                            on:click={() => fileInput?.click()}
                        >
                            <img 
                                src={newProfilePicture || $userProfile.profilePicture} 
                                alt="Preview" 
                                class="w-full h-full object-cover transition-transform duration-300 group-hover:scale-110"
                            />
                            <div class="absolute inset-0 bg-black bg-opacity-0 group-hover:bg-opacity-30 flex items-center justify-center transition-all duration-300">
                                <svg 
                                    xmlns="http://www.w3.org/2000/svg" 
                                    class="h-8 w-8 text-white opacity-0 group-hover:opacity-100 transition-opacity duration-300" 
                                    fill="none" 
                                    viewBox="0 0 24 24" 
                                    stroke="currentColor"
                                >
                                    <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M3 9a2 2 0 012-2h.93a2 2 0 001.664-.89l.812-1.22A2 2 0 0110.07 4h3.86a2 2 0 011.664.89l.812 1.22A2 2 0 0018.07 7H19a2 2 0 012 2v9a2 2 0 01-2 2H5a2 2 0 01-2-2V9z" />
                                    <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M15 13a3 3 0 11-6 0 3 3 0 016 0z" />
                                </svg>
                            </div>
                        </div>
                        <input 
                            type="file" 
                            bind:this={fileInput}
                            on:change={handleFileSelect} 
                            accept="image/*" 
                            class="hidden"
                        />
                    </div>

                    <div>
                        <label class="block text-sm font-medium text-gray-700 mb-2">Username</label>
                        <input
                            type="text"
                            bind:value={newUserName}
                            class="mt-1 block w-full rounded-md border-gray-300 shadow-sm focus:border-[#FF4500] focus:ring-2 focus:ring-[#FF4500]/50 transition-all duration-300"
                            placeholder="Enter your username"
                        />
                    </div>

                    <div>
                        <label class="block text-sm font-medium text-gray-700 mb-2">Moods (Select Multiple)</label>
                        <div class="mt-2 flex justify-center flex-wrap gap-2">
                            {#each moods as mood}
                                <button
                                    class="px-4 py-2 rounded-full text-sm font-medium transition-all duration-300 
                                    {selectedMoods.includes(mood.value) 
                                        ? 'bg-[#FF4500] text-white ring-2 ring-[#FF4500]' 
                                        : 'bg-gray-100 text-gray-600 hover:bg-gray-200'}"
                                    on:click={() => {
                                        if (selectedMoods.includes(mood.value)) {
                                            selectedMoods = selectedMoods.filter(m => m !== mood.value);
                                        } else {
                                            selectedMoods = [...selectedMoods, mood.value];
                                        }
                                    }}
                                >
                                    {mood.value}
                                </button>
                            {/each}
                        </div>
                        {#if selectedMoods.length > 0}
                            <p class="text-xs text-gray-500 text-center mt-2">
                                Selected: {selectedMoods.join(', ')}
                            </p>
                        {/if}
                    </div>
                </div>

                <div class="mt-6 flex justify-center space-x-3">
                    <button
                        on:click={() => showEditModal = false}
                        class="px-4 py-2 text-gray-600 hover:bg-gray-100 rounded-md transition-colors duration-300"
                    >
                        Cancel
                    </button>
                    <button
                        on:click={saveProfileChanges}
                        class="px-4 py-2 bg-[#FF4500] text-white rounded-md hover:bg-[#FF5722] transition-colors duration-300 transform active:scale-95"
                    >
                        Save Changes
                    </button>
                </div>
            </div>
        </div>
    </div>
{/if}

<style>
    /* For Firefox masonry grid support */
    @-moz-document url-prefix() {
        .columns-1 { column-count: 1; }
        .columns-2 { column-count: 2; }
        .columns-3 { column-count: 3; }
        .columns-4 { column-count: 4; }
    }
</style>