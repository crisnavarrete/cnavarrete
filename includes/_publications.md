```{python}
import yaml
from IPython.display import display, Markdown, HTML

def readable_list(_s):
  if len(_s) < 3:
    return ' and '.join(map(str, _s))
  *a, b = _s
  return f"{', '.join(map(str, a))}, and {b}"

def button(url, str, icon):
    icon_base = icon[:2]
    return f"""
    <a class="btn btn-custom", href="{url}" target="_blank" rel="noopener noreferrer">
        <i class="{icon_base} {icon}" role='img' aria-label='{str}'></i>
        {str}
    </a>
    """


yaml_data = yaml.safe_load(open("papers.yaml", encoding= "utf-8"))
pub_strs = {"pubs": {}, "wps": {}}
for _, data in yaml_data.items():
    title_str = data["title"]
    authors = data.get("authors", ["me"])
    authors = [aut if aut != "me" else "<strong style='color: #01B587 !important;'>Navarrete, C</strong>" for aut in authors]
    author_str = readable_list(authors)
    year_str = data["year"]

    buttons = []
    preprint = data.get("preprint")
    if preprint is not None:
        buttons.append(button(preprint, "Go to preprint", "bi-file-earmark-pdf"))

    github = data.get("github")
    if github is not None:
        buttons.append(button(github, "Github", "bi-github"))

    pub_url = data.get("published_url")
    if pub_url is not None:
        buttons.append(button(pub_url, "Go to published version", "bi-file-earmark-pdf"))
        
    venue = data.get("venue")
    working_paper = pub_url is None
    
    pub_str = f'{author_str}. ({year_str}) "{title_str}."'

    if venue is not None:
        pub_str += f" <em>{venue}</em>"

    if working_paper:
        if year_str not in pub_strs["wps"]:
            pub_strs["wps"][year_str] = []
        pub_strs["wps"][year_str].append(
            "<li class='list-group-item'>" + pub_str + "<br>" + " ".join(buttons) + "</li>"
        )
    else:
        if year_str not in pub_strs["pubs"]:
            pub_strs["pubs"][year_str] = []
        pub_strs["pubs"][year_str].append(
            "<li class='list-group-item'>" + pub_str + "<br>" + " ".join(buttons) + "</li>"
        )
```

```{python}
#| label: "published-year"
#| id: "published-year"
#| output: asis
display(Markdown("## Published"))
for year in sorted(pub_strs["pubs"].keys(), reverse=True):
    display(Markdown(f"### {year}" + "{#" + f"published-{year}" + "}"))
    display(HTML(
        "<ul class='list-group list-group-flush'>" + '\n'.join(pub_strs["pubs"][year]) + "</ul>"
    ))
```

```{python}
#| label: "working-papers-year"
#| id: "working-papers-year"
#| output: asis
display(Markdown("## Working Papers"))
for year in sorted(pub_strs["wps"].keys(), reverse=True):
    display(Markdown(f"### {year}" + "{#" + f"working-papers-{year}" + "}"))
    display(HTML(
        "<ul class='list-group list-group-flush'>" + '\n'.join(pub_strs["wps"][year]) + "</ul>"
    ))
```