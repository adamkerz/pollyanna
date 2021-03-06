# pollyanna

The Plan, as it really is.

You may be familiar with the famous and most excellent "Kanned Bananas",
which draws data from GitHub (using it's GraphQL API)
and ZehHub (using it's REST API),
and produces reports of **what actually happened**
as well as **what work is currently in progress**.
If not, you should check it out and probably start using it.

Pollyanna is similar, in that it draws data from GitHub GraphQL API and ZenHub REST API,
but different in that it documents **what the plan says is going to happen**.
Kanned Banannas answers the question "between these dates, what happened?".
Pollyanna answers the question "based on everything we know up until now,
what are we saying we are going to do?".

See, similar but different. Maybe you need both?


## Diffable Output

Pollyanna generates a diffable output.

This means that when the tickets change,
and you keep clobbering you old planning documents with new ones,
the only difference will be the things that changed.
So you can put your generated documents under version control
and use `git diff` to see how the plan changed over time.

This means that each change (commit) is readable,
it has a high signal to noise ratio.
It should not require a big cognative investment
to understand changes by looking at them.

This is because the the plan is an important artefact,
and changes to the plan are important events.

## Consolodated plan

The plan needs to be distributed among tickets
so that you can do it,
but it also needs to be consolidated in one place
so that you can present it to people
who need to know what the plan is.

This means more than just the people
who need to do what is in the plan.
They are generally happy with tickets at home,
"in the context of doing".
But other stakeholders need to work down to the detail
from their perspective.

You need to be able to discuss how the plan has changed,
why the plan changed, and when.

## Install

Do this:
```
virtualenv -p python3.8 .venv
. .venv/bin/activate
pip install -r requirements.txt
./post_pip_install.sh  # because we want bleeding edge of gql
cp local_settings.py.sample local_settings.py
pytest
```

The last thing you should see is something like this:
```
=================================== test session starts ====================================
platform linux -- Python 3.8.0, pytest-6.1.2, py-1.9.0, pluggy-0.13.1
rootdir: /home/chris/src/supman/supman/docs/jupyter_mashup, configfile: pytest.ini
plugins: mock-3.3.1
collected 11 items                                                                         

test_fs_cache.py .......                                                             [ 63%]
test_repos.py ...                                                                    [ 90%]
test_utils.py .                                                                      [100%]

==================================== 11 passed in 0.14s ====================================
```
But hopefully with more tests :)

Then copy `local_settings.py.sample` to `local_settings.py` and edit it.
It only needs to exist for the tests to run,
but you need to configure it for your project to get a sensible output.

Then do `python run.py` or, if you are lucky just `run.py` should work for you.

## How it (is going to) works

run.py instantiates a `usecases.GenerateThePlan` instance,
feeding it some handy "repository" instances.
These are repositories in the Clean Architecture sense of the word,
not Git Repositories.

When `run.py` does it's thing,
the `repositories.ZenHubRestRepo`
and `repositories.GitHubGraphQPRepo` instances
talk to the online services and get the domain objects we need.
These repos use a `repositories.FSCache` class (File System Cache)
to avoid hammering the backing services.

to form a linked-up object graph of domain objects.
This graph is passed to a `repositories.RSTPlanRepo` instance,
which uses jinja2 templates to create
a tree of ReStructured Text (rst) documentation.

This generated rst is the "diffable" product,
which may be kept in a version control system.
It uses ReStructured Text *directives* to generate diagrams and artefacts.

Those generated RST files will make use of cool stuff at render-time,
such as https://sphinxcontrib-needs.readthedocs.io/.
So the generated/version-controlled stuff is hightly diffable,
but the rendered/published/distributed stuff
is hightl consumable as-is (rather than as-a-delta).

Maybe even one day, there could be a dynamic (render-time)
appendix containing an introspected change-log.
e.g. using Ned Beltcher's Cog to query the git history.
