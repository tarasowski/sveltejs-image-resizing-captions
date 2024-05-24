<script>
    import QuillResizeImage from "quill-resize-image";
    import Quill from "quill";
    import { onMount } from "svelte";

    let editor;
    let quill;

    function imageHandler() {
        let fileInput = this.container.querySelector(
            "input.ql-image[type=file]",
        );
        if (fileInput == null) {
            fileInput = document.createElement("input");
            fileInput.setAttribute("type", "file");
            fileInput.setAttribute(
                "accept",
                "image/png, image/gif, image/jpeg, image/bmp, image/x-icon",
            );
            fileInput.classList.add("ql-image");
            fileInput.addEventListener("change", () => {
                const files = fileInput.files;
                const range = this.quill.getSelection(true);

                if (files != null && files[0] != null) {
                    const reader = new FileReader();
                    reader.onloadend = () => {
                        // Prompt the user for the caption
                        const caption = prompt(
                            "Enter the caption for the image:",
                        );
                        this.quill.insertEmbed(range.index, "imageCaption", {
                            url: reader.result,
                            alt: caption,
                        });
                        this.quill.setSelection(
                            range.index + 1,
                            Quill.sources.SILENT,
                        );
                    };
                    reader.readAsDataURL(files[0]);
                }
            });
            this.container.appendChild(fileInput);
        }
        fileInput.click();
    }

    onMount(async () => {
        //@ts-ignore
        let BlockEmbed = Quill.import("blots/block/embed");
        class ImageCaptionBlot extends BlockEmbed {
            static create(value) {
                let node = super.create();
                let img = document.createElement("img");
                let figcaption = document.createElement("figcaption");

                img.setAttribute("src", value.url);
                figcaption.innerText = value.alt;

                node.appendChild(img);
                node.appendChild(figcaption);

                return node;
            }

            static value(node) {
                let img = node.querySelector("img");
                let figcaption = node.querySelector("figcaption");
                return {
                    url: img.getAttribute("src"),
                    alt: figcaption.innerText,
                };
            }
        }
        ImageCaptionBlot.blotName = "imageCaption";
        ImageCaptionBlot.tagName = "figure";

        Quill.register(ImageCaptionBlot);
        Quill.register("modules/imageResize", QuillResizeImage);
        quill = new Quill(editor, {
            theme: "snow",
            modules: {
                imageResize: {
                    displaySize: true,
                },
                toolbar: {
                    container: [
                        [{ header: [1, 2, 3, 4, 5, 6, false] }],
                        ["bold", "italic", "underline", "strike"],
                        [{ color: [] }, { background: [] }],
                        [{ align: [] }],
                        ["link", "image"],
                        [],
                        ["clean"],
                    ],
                    handlers: {
                        image: imageHandler,
                    },
                },
            },
        });
    });
</script>

<h1>hello from quill</h1>
<div id="editor" style="width: 800px; height: 800px" bind:this={editor}></div>

<style>
    @import "https://cdn.quilljs.com/1.3.6/quill.snow.css";
    :global(::selection) {
        background-color: #475569;
        color: white;
    }

    :global(.ql-editor) {
        font-size: 16px;
        background-color: white;
        border-radius: 3px;
        min-height: 100vh;
        padding: 13px;
    }

    :global(.ql-toolbar) {
        background-color: white !important;
        border: none !important;
        position: sticky !important;
        top: 0 !important;
        z-index: 0;
    }

    :global(.ql-snow h1) {
        font-size: 1.6rem !important;
        font-weight: bold !important;
    }

    :global(.ql-snow h2) {
        font-size: 1.4rem !important;
        font-weight: bold !important;
    }

    :global(.ql-snow h3) {
        font-size: 1.2rem !important;
        font-weight: bold !important;
    }

    :global(.ql-clipboard) {
        position: fixed;
        display: none;
        left: 50%;
        top: 50%;
    }

    .nav-tabs .nav-item .nav-link {
        background-color: #0080ffcc !important;
    }

    .modal-footer {
        background-color: #f9fafb !important;
    }
    :global(figcaption) {
        padding-top: 10px;
    }
</style>
