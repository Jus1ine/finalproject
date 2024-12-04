<script lang="ts">
    import { onMount } from 'svelte';
    import { goto } from '$app/navigation';

    interface GalleryImage {
        id: number;
        url: string;
        name: string;
        timestamp: string;
    }

    interface ImageFilters {
        brightness: number;
        contrast: number;
        saturation: number;
        sharpness: number;
        grain: number;
        blur: number;
        hue: number;
        sepia: number;
        grayscale: number;
    }

    let showModal = false;
    let images: GalleryImage[] = [];
    let selectedFiles: File[] = [];
    let previewUrls: string[] = [];
    let showEditMode = false;
    let selectedImageForEdit: GalleryImage | null = null;
    
    let filters: ImageFilters = {
        brightness: 100,
        contrast: 100,
        saturation: 100,
        sharpness: 100,
        grain: 0,
        blur: 0,
        hue: 0,
        sepia: 0,
        grayscale: 0
    };

    onMount(() => {
        const storedImages = localStorage.getItem('galleryImages');
        if (storedImages) {
            images = JSON.parse(storedImages) as GalleryImage[];
        }
    });

    function toggleModal() {
        showModal = !showModal;
        if (!showModal) {
            resetFileSelection();
        }
    }

    function resetFileSelection() {
        selectedFiles = [];
        previewUrls = [];
    }

    function handleFileSelect(event: Event) {
        const input = event.target as HTMLInputElement;
        if (input.files) {
            const newFiles = Array.from(input.files);
            addFiles(newFiles);
        }
    }

    function handleDrop(event: DragEvent) {
        event.preventDefault();
        if (event.dataTransfer?.files) {
            const newFiles = Array.from(event.dataTransfer.files)
                .filter(file => file.type.startsWith('image/'));
            addFiles(newFiles);
        }
    }

    function addFiles(newFiles: File[]) {
        // Limit total files to 10
        const remainingSlots = Math.max(0, 10 - selectedFiles.length);
        const filesToAdd = newFiles.slice(0, remainingSlots);

        filesToAdd.forEach(file => {
            selectedFiles = [...selectedFiles, file];
            previewUrls = [...previewUrls, URL.createObjectURL(file)];
        });
    }

    function removeFile(index: number) {
        selectedFiles = selectedFiles.filter((_, i) => i !== index);
        previewUrls = previewUrls.filter((_, i) => i !== index);
    }

    function uploadImages() {
        if (selectedFiles.length === 0) return;

        selectedFiles.forEach(file => {
            const reader = new FileReader();
            reader.onload = (e: ProgressEvent<FileReader>) => {
                const result = e.target?.result;
                if (result && typeof result === 'string') {
                    const newImage: GalleryImage = {
                        id: Date.now() + Math.random(),
                        url: result,
                        name: file.name,
                        timestamp: new Date().toISOString()
                    };
                    images = [...images, newImage];
                }
            };
            reader.readAsDataURL(file);
        });

        localStorage.setItem('galleryImages', JSON.stringify(images));
        toggleModal();
    }

    function openEditMode(image: GalleryImage) {
        selectedImageForEdit = image;
        showEditMode = true;
        resetFilters();
    }

    function closeEditMode() {
        showEditMode = false;
        selectedImageForEdit = null;
        resetFilters();
    }

    function resetFilters() {
        filters = {
            brightness: 100,
            contrast: 100,
            saturation: 100,
            sharpness: 100,
            grain: 0,
            blur: 0,
            hue: 0,
            sepia: 0,
            grayscale: 0
        };
    }

    function getFilterStyle(): string {
        return `
            filter: 
                brightness(${filters.brightness}%) 
                contrast(${filters.contrast}%) 
                saturate(${filters.saturation}%)
                blur(${filters.blur}px)
                hue-rotate(${filters.hue}deg)
                sepia(${filters.sepia}%)
                grayscale(${filters.grayscale}%)
                opacity(${100}%)
        `;
    }

    function saveEditedImage() {
        // Non-null assertion and type guard
        if (!selectedImageForEdit) {
            console.error('No image selected for editing');
            return;
        }

        try {
            // Create a canvas to apply filters
            const canvas = document.createElement('canvas');
            const ctx = canvas.getContext('2d');
            
            if (!ctx) {
                console.error('Could not get canvas context');
                return;
            }

            const img = new Image();
            img.crossOrigin = 'anonymous'; // Help with potential CORS issues
            img.src = selectedImageForEdit.url;

            img.onload = () => {
                // Set canvas size to match image
                canvas.width = img.width;
                canvas.height = img.height;

                // Apply filters using canvas
                ctx.filter = `
                    brightness(${filters.brightness}%) 
                    contrast(${filters.contrast}%) 
                    saturate(${filters.saturation}%)
                    blur(${filters.blur}px)
                    hue-rotate(${filters.hue}deg)
                    sepia(${filters.sepia}%)
                    grayscale(${filters.grayscale}%)
                `;

                // Draw the filtered image
                ctx.drawImage(img, 0, 0, img.width, img.height);

                // Convert canvas to data URL
                const filteredImageUrl = canvas.toDataURL('image/jpeg', 0.9);

                // Update the image in the images array
                images = images.map(image => 
                    image.id === selectedImageForEdit!.id 
                        ? { ...image, url: filteredImageUrl } 
                        : image
                );

                // Update localStorage
                try {
                    localStorage.setItem('galleryImages', JSON.stringify(images));
                } catch (storageError) {
                    console.error('Error saving to localStorage:', storageError);
                }

                // Close edit mode
                closeEditMode();
            };

            img.onerror = (error) => {
                console.error('Error loading image:', error);
            };
        } catch (error) {
            console.error('Error in saveEditedImage:', error);
        }
    }

    function deleteImage(id: number) {
        images = images.filter(img => img.id !== id);
        localStorage.setItem('galleryImages', JSON.stringify(images));
    }

    function navigateToUrImage() {
        goto('/UrImage');
    }
</script>

<div class="container mx-auto p-4 relative">
    <div class="flex justify-between items-center mb-6">
        <div class="flex items-center gap-4">
            <h1 class="text-3xl font-bold">Gallery</h1>
            <button 
                on:click={toggleModal}
                class="px-4 py-2 bg-blue-500 text-white rounded-full shadow-lg hover:bg-blue-600 transition-colors flex items-center gap-2"
            >
                <svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                    <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 4v16m8-8H4" />
                </svg>
                Upload Images
            </button>
        </div>
        <div class="flex items-center gap-4">
            <button 
                on:click={navigateToUrImage}
                class="px-4 py-2 bg-green-500 text-white rounded-full shadow-lg hover:bg-green-600 transition-colors flex items-center gap-2"
            >
                <svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                    <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M4 6h16M4 10h16M4 14h16M4 18h16" />
                </svg>
                Your Images
            </button>
        </div>
    </div>

    <!-- Upload Modal -->
    {#if showModal}
        <div class="fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center z-50 p-4">
            <div 
                class="bg-white rounded-lg w-full max-w-2xl max-h-[90vh] flex flex-col"
                on:dragover|preventDefault
                on:drop|preventDefault={(event: DragEvent) => handleDrop(event)}
                role="region"
                aria-label="Image upload dropzone"
            >
                <!-- Modal Header -->
                <div class="p-4 border-b flex justify-between items-center">
                    <h2 class="text-xl font-semibold" id="modal-title">Upload Images</h2>
                    <button 
                        on:click={toggleModal}
                        class="text-gray-500 hover:text-gray-700"
                        aria-label="Close upload modal"
                    >
                        <svg class="w-6 h-6" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M6 18L18 6M6 6l12 12" />
                        </svg>
                    </button>
                </div>

                <!-- Drag and Drop / File Input Area -->
                <div class="p-6 flex-grow overflow-y-auto">
                    <div 
                        class="border-2 border-dashed rounded-lg p-6 text-center transition-colors 
                        {selectedFiles.length > 0 ? 'border-blue-300 bg-blue-50' : 'border-gray-300 hover:border-blue-300'}"
                    >
                        {#if selectedFiles.length === 0}
                            <label for="fileInput" class="block mb-4 text-gray-600">
                                Drag & drop images here or click to select
                            </label>
                            <input
                                type="file"
                                id="fileInput"
                                accept="image/*"
                                multiple
                                on:change={handleFileSelect}
                                class="hidden"
                            />
                            <button
                                on:click={() => document.getElementById('fileInput')?.click()}
                                class="px-4 py-2 bg-blue-500 text-white rounded-md hover:bg-blue-600"
                                aria-label="Select images to upload"
                            >
                                Select Images
                            </button>
                        {:else}
                            <div class="grid grid-cols-2 md:grid-cols-3 gap-4">
                                {#each previewUrls as preview, index}
                                    <div class="relative">
                                        <img 
                                            src={preview} 
                                            alt={`Preview ${index + 1}`} 
                                            class="w-full h-32 object-cover rounded"
                                        />
                                        <button 
                                            on:click={() => removeFile(index)}
                                            class="absolute top-2 right-2 bg-red-500 text-white rounded-full w-6 h-6 flex items-center justify-center hover:bg-red-600"
                                            aria-label={`Remove image ${index + 1}`}
                                        >
                                            ×
                                        </button>
                                    </div>
                                {/each}
                                {#if selectedFiles.length < 10}
                                    <div class="relative">
                                        <label 
                                            for="fileInput" 
                                            class="flex items-center justify-center w-full h-32 border-2 border-dashed rounded cursor-pointer hover:bg-gray-50"
                                        >
                                            <svg class="w-8 h-8 text-gray-400" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                                                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 4v16m8-8H4" />
                                            </svg>
                                        </label>
                                        <input
                                            type="file"
                                            id="fileInput"
                                            accept="image/*"
                                            multiple
                                            on:change={handleFileSelect}
                                            class="hidden"
                                        />
                                    </div>
                                {/if}
                            </div>
                        {/if}
                    </div>
                </div>

                <!-- Modal Footer -->
                <div class="p-4 border-t flex justify-end gap-3">
                    <button 
                        on:click={toggleModal}
                        class="px-4 py-2 text-gray-600 hover:bg-gray-100 rounded-md"
                    >
                        Cancel
                    </button>
                    <button 
                        on:click={uploadImages}
                        disabled={selectedFiles.length === 0}
                        class="px-6 py-2 bg-blue-500 text-white rounded-md hover:bg-blue-600 disabled:opacity-50 disabled:cursor-not-allowed"
                    >
                        Upload {selectedFiles.length} Image{selectedFiles.length !== 1 ? 's' : ''}
                    </button>
                </div>
            </div>
        </div>
    {/if}

    <!-- Edit Mode -->
    {#if showEditMode && selectedImageForEdit}
        <div class="fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center z-50">
            <div class="bg-white rounded-lg w-[90vw] max-w-5xl h-[90vh] flex flex-col">
                <!-- Header -->
                <div class="flex justify-between items-center p-4 border-b">
                    <h2 class="text-xl font-semibold" id="edit-mode-title">Edit Image</h2>
                    <button 
                        on:click={closeEditMode}
                        class="text-gray-500 hover:text-gray-700"
                        aria-label="Close edit mode"
                    >
                        <svg class="w-6 h-6" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M6 18L18 6M6 6l12 12" />
                        </svg>
                    </button>
                </div>

                <!-- Content -->
                <div class="flex flex-1 overflow-hidden">
                    <!-- Image Preview -->
                    <div class="flex-1 bg-[#f8f9fa] p-6 overflow-hidden">
                        <div class="w-full h-full flex items-center justify-center">
                            <img 
                                src={selectedImageForEdit.url} 
                                alt={selectedImageForEdit.name}
                                class="max-w-full max-h-full object-contain rounded-lg shadow-lg"
                                style={getFilterStyle()}
                            >
                        </div>
                    </div>

                    <!-- Controls -->
                    <div class="w-80 border-l">
                        <div class="p-6 h-full overflow-y-auto">
                            <h3 class="text-lg font-medium mb-6" id="adjustments">Adjustments</h3>
                            
                            <!-- Basic Adjustments -->
                            <div class="mb-8">
                                <h4 class="text-sm font-medium text-gray-700 mb-4" id="basic-adjustments">Basic Adjustments</h4>
                                <div class="space-y-6">
                                    <div>
                                        <label for="brightness" class="flex justify-between mb-2">
                                            <span>Brightness</span>
                                            <span>{filters.brightness}%</span>
                                        </label>
                                        <input
                                            type="range"
                                            id="brightness"
                                            min="0"
                                            max="200"
                                            bind:value={filters.brightness}
                                            class="w-full"
                                        />
                                    </div>

                                    <div>
                                        <label for="contrast" class="flex justify-between mb-2">
                                            <span>Contrast</span>
                                            <span>{filters.contrast}%</span>
                                        </label>
                                        <input
                                            type="range"
                                            id="contrast"
                                            min="0"
                                            max="200"
                                            bind:value={filters.contrast}
                                            class="w-full"
                                        />
                                    </div>

                                    <div>
                                        <label for="saturation" class="flex justify-between mb-2">
                                            <span>Saturation</span>
                                            <span>{filters.saturation}%</span>
                                        </label>
                                        <input
                                            type="range"
                                            id="saturation"
                                            min="0"
                                            max="200"
                                            bind:value={filters.saturation}
                                            class="w-full"
                                        />
                                    </div>

                                    <div>
                                        <label for="blur" class="flex justify-between mb-2">
                                            <span>Blur</span>
                                            <span>{filters.blur}px</span>
                                        </label>
                                        <input
                                            type="range"
                                            id="blur"
                                            min="0"
                                            max="10" 
                                            step="0.1"
                                            bind:value={filters.blur}
                                            class="w-full"
                                        />
                                    </div>
                                </div>
                            </div>

                            <!-- Color Adjustments -->
                            <div class="mb-8">
                                <h4 class="text-sm font-medium text-gray-700 mb-4" id="color-adjustments">Color Adjustments</h4>
                                <div class="space-y-6">
                                    <div>
                                        <label for="hue" class="flex justify-between mb-2">
                                            <span>Hue</span>
                                            <span>{filters.hue}°</span>
                                        </label>
                                        <input
                                            type="range"
                                            id="hue"
                                            min="0"
                                            max="360"
                                            bind:value={filters.hue}
                                            class="w-full"
                                        />
                                    </div>

                                    <div>
                                        <label for="sepia" class="flex justify-between mb-2">
                                            <span>Sepia</span>
                                            <span>{filters.sepia}%</span>
                                        </label>
                                        <input
                                            type="range"
                                            id="sepia"
                                            min="0"
                                            max="100"
                                            bind:value={filters.sepia}
                                            class="w-full"
                                        />
                                    </div>

                                    <div>
                                        <label for="grayscale" class="flex justify-between mb-2">
                                            <span>Grayscale</span>
                                            <span>{filters.grayscale}%</span>
                                        </label>
                                        <input
                                            type="range"
                                            id="grayscale"
                                            min="0"
                                            max="100"
                                            bind:value={filters.grayscale}
                                            class="w-full"
                                        />
                                    </div>
                                </div>
                            </div>

                            <!-- Action Buttons -->
                            <div class="flex gap-3">
                                <button 
                                    on:click={resetFilters}
                                    class="flex-1 px-4 py-2 text-sm font-medium text-gray-700 bg-gray-100 rounded-md hover:bg-gray-200 transition-colors"
                                >
                                    Reset
                                </button>
                                <button 
                                    on:click={saveEditedImage}
                                    class="flex-1 px-4 py-2 text-sm font-medium text-white bg-blue-500 rounded-md hover:bg-blue-600 transition-colors"
                                >
                                    Save
                                </button>
                            </div>
                        </div>
                    </div>
                </div>
            </div>
        </div>
    {/if}

    <!-- Gallery Grid -->
    {#if images.length === 0}
        <div class="text-center text-gray-500 py-8">
            <p>No images uploaded yet. Click the Upload button to add some!</p>
        </div>
    {:else}
        <div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-4">
            {#each images as image (image.id)}
                <div class="relative group">
                    <img 
                        src={image.url} 
                        alt={image.name}
                        class="w-full h-64 object-cover rounded-lg shadow-md"
                    />
                    <div class="absolute inset-0 bg-black bg-opacity-0 group-hover:bg-opacity-50 transition-all duration-300 rounded-lg flex items-center justify-center gap-2">
                        <button 
                            class="opacity-0 group-hover:opacity-100 px-4 py-2 bg-white text-black rounded hover:bg-gray-100 transition-all duration-300"
                            on:click={() => openEditMode(image)}
                            aria-label="Edit image"
                        >
                            Edit Image
                        </button>
                        <button 
                            class="opacity-0 group-hover:opacity-100 px-4 py-2 bg-red-500 text-white rounded hover:bg-red-600 transition-all duration-300"
                            on:click={() => deleteImage(image.id)}
                            aria-label="Delete image"
                        >
                            Delete
                        </button>
                    </div>
                    <div class="mt-2">
                        <p class="text-sm text-gray-600">{image.name}</p>
                        <p class="text-xs text-gray-400">{new Date(image.timestamp).toLocaleDateString()}</p>
                    </div>
                </div>
            {/each}
        </div>
    {/if}
</div>