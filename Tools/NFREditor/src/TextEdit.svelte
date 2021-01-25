<script lang="ts">
    export let path: string;
    let currentFile: GitFile;
    let editor: Jodit.IJodit;
    let title: string;
    let tagString: string;
    let tags: string[];

    import APIHelper from "./api";
    import { onMount } from "svelte";
    import Jodit from "jodit";
    import type GitFile from "./gitFile";
    

    $: {
        console.log("Loading: " + path);
        APIHelper.getFileContent(path).then((file) => {
            currentFile = file;
            initJodit();
        });
    }
    onMount(() => {
        console.log("Loading: " + path);
        APIHelper.getFileContent(path).then((file) => {
            currentFile = file;
            initJodit();
        });
    });
    function initJodit() {
        console.log("initJodit");
        if (!editor) {
            editor = new Jodit.Jodit("#editor", {
                uploader: {
                    insertImageAsBase64URI: true,
                },

                extraButtons: [
                    {
                        name: "insertDate",
                        icon: "save",
                        tooltip: "Save and commit",
                        exec: function (editor) {
                            //editor.s.insertHTML(new Date().toDateString());
                            console.log(editor.value);
                            saveAndCommit();
                        },
                    },
                ],
                "beautify-html": true,
            });
        }
        let content = atob(currentFile.content);
        let parts = extractHTMLParts(content);
        title = parts[0].toString();
        editor.value = parts[1].toString();
    }

    function getElementFromText(content: string): HTMLHtmlElement {
        var el = document.createElement("html");
        el.innerHTML =
            "<html><head><title></title></head><body>" +
            content +
            "</body></html>";
        return el;
    }

    function extractHTMLParts(content: string) {
        var el = document.createElement("html");
        el.innerHTML =
            "<html><head><title></title></head><body>" +
            content +
            "</body></html>";

        let col = el.getElementsByClassName("description");
        let description = col[0] ? col[0].innerHTML : " ";
        console.log("Description: " + description);

        col = el.getElementsByClassName("title");
        let title = col[0] ? col[0].innerHTML : " ";
        console.log("Title: " + title);

        let tags = [];
        col = el.getElementsByClassName("tags");
        if (col[0]) {
            let tagElement = col[0].getElementsByClassName("tag");

            for (let i = 0; i < tagElement.length; ++i) {
                tags.push(tagElement[i].innerHTML);
            }
            tagString = tags.join(",");
        }
        return [title, description, tags];
    }

    function saveAndCommit() {
        generateHTML().then((html) => {
         
            //@ts-ignore
            html= html_beautify(html, { indent_size: 2, space_in_empty_paren: true } );
         
            
            currentFile.content = html;

            APIHelper.writeFile(currentFile).then((file) => {
                console.log("File committed");
                currentFile.sha = file.sha;
            });
        });
    }

    function generateHTML(): Promise<string> {
        return new Promise<string>((resolve, reject) => {
            fetch("htmlTemplate.html")
                .then((response) => response.text())
                .then((text) => {
                    let el = getElementFromText(text);
                    
                    el.getElementsByClassName("title")[0].innerHTML = title;
                    el.getElementsByClassName("description")[0].innerHTML =
                        editor.value;

                    tags = tagString.split(",");
                    let tagHTML = "";
                    tags.forEach((val,idx) =>{
                        tagHTML += `<div class='tag'>${val}</div>`;
                    });
                    el.getElementsByClassName("tags")[0].innerHTML = tagHTML;

                    let res = el.getElementsByTagName("body")[0].innerHTML;

                    console.log("Saved HTML: " + res);
                    console.log("Saved tags: " + tags);
                    
                    resolve(res);
                });
        });
    }
</script>

<main>
    <input bind:value={title} />
    <input bind:value={tagString} />
    <textarea id="editor" name="editor" />
</main>
<svelte:head>
    <link type="text/css" rel="stylesheet" href="build/jodit.min.css" />
</svelte:head>

<style>
    #editor {
        min-width: 400px;
        min-height: 400px;
    }

    input{
		min-width: 500px;
		font-size: smaller;
	}
</style>
