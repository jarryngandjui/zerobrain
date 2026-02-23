<%*
const quickAddApi = app.plugins.plugins.quickadd.api;
const file = tp.config.target_file;


async function setBig7(frontmatter) {
    frontmatter["kind"] = ["sprint"];
    frontmatter["category"] = ["planning"];
    frontmatter["tags"] = [];
}

async function setWorkout(frontmatter) {
    frontmatter["kind"] = ["sprint"];
    frontmatter["tags"] = ["gym", "road-to-dunk"];
    frontmatter["category"] = ["health"];
}

async function setContent(frontmatter) {
    frontmatter["kind"] = ["story"];
    frontmatter["tags"] = ["content/instagram", "content/blog"];
    frontmatter["category"] = ["brand"];
    frontmatter["project"] = ["[[Brand]]"];
}

async function setBlog(frontmatter) {
    frontmatter["kind"] = ["story"];
    frontmatter["tags"] = ["content/blog", "website"];
    frontmatter["category"] = ["brand"];
	frontmatter["project"] = ["[[Brand]]"];
}

async function setPeople(frontmatter) {
    frontmatter["kind"] = ["story"];
    frontmatter["tags"] = [];
    frontmatter["category"] = ["social"];
}


const options = ["content", "big7", "blog", "people", "workout"];
const choice = await quickAddApi.suggester(options, options);

if (!choice) return;

await app.fileManager.processFrontMatter(file, (frontmatter) => {


    // Switch to the specific function based on selection
    switch (choice) {
        case "big7":
            setBig7(frontmatter);
            break;
		case "workout":
            setWorkout(frontmatter);
            break;
        case "blog":
            setBlog(frontmatter);
            break;
		case "content":
            setContent(frontmatter);
            break;
        case "people":
            setPeople(frontmatter);
            break;

    }
});

new Notice(`Kind updated to: ${choice}`);
%>