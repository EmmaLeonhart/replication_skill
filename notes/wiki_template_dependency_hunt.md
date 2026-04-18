# Wiki template deep-clone dependency hunt — notes

> Source: Claude chat (HTML title: "Wiki efn template lower alpha display"). Kept because the pattern at the end — deep clones miss TemplateStyles pages because they aren't in the transclusion tree — parallels how arXiv replications miss hidden dependencies (unpinned libs, dataset versions, system tools).

on a wiki with imported efn template it says lower alpha

---

This is a classic MediaWiki/Wikipedia template issue. When you import the
{{efn}}
template, it uses a list-defined references group called
lower-alpha
, but the associated
{{notelist}}
(or
<references group="lower-alpha"/>
) isn't rendering the footnotes with the correct styling — so instead of showing
a, b, c...
you see the literal group name "lower-alpha" next to the reference markers.
The fix depends on what's missing:
Most common cause: missing
{{notelist}}
or wrong reflist call
Make sure you're using
{{notelist}}
at the bottom of the page, not plain
<references/>
. The
efn
template outputs to the
lower-alpha
group, and
{{notelist}}
knows to collect that group with alphabetic styling.
If you imported
efn
but not its dependencies
{{efn}}
depends on
{{efn/sandbox}}
or more importantly on
{{reflist}}
and the CSS/Lua module chain. Check that you also imported:
{{notelist}}
{{reflist}}
Module:Footnotes
(the Lua module that actually powers both)
If the Lua module is missing
, the template falls back to basic
<ref group="lower-alpha">
behavior, which just prints the group name literally.
Quick checklist:
Did you import
Module:Footnotes
along with the template?
Are you calling
{{notelist}}
(not
<references/>
) where the notes should appear?
Did you also import
{{reflist}}
since
notelist
calls it internally?
If you share which wiki software version you're on or what exactly you did import, I can narrow it down further.

---

Can  you cheat with something else like a parameter

---

Yes — if you just want alphabetic footnotes without the full template dependency chain, you can use the
group
parameter on plain
<ref>
tags with a custom group name, and then style it yourself. But the simplest cheat is:
<ref group="note">Your footnote text</ref>
And at the bottom:
<references group="note"/>
That gives you
note 1, note 2...
which avoids the "lower-alpha" display problem entirely, just with a different label style.
The actual cheat for alphabetic display
is to use
{{efn}}
but override the group name it uses by passing
|group=
explicitly... except
efn
doesn't expose that.
The real minimal cheat is
{{efn-la}}
if your wiki has it — it's the explicit lower-alpha variant.
Or the dirtiest cheat:
just hardcode the letter in the footnote marker using a named ref:
<ref name="a">Your note</ref>
Not truly automatic, but if you only have a few footnotes it sidesteps the whole problem.
What's the actual goal — do you want the
a b c
markers showing up on the citations in the text, or is it more that the rendered notelist just needs to look right? That'll determine which shortcut actually fits.

---

Rendered notelist look right

---

For just fixing the rendered notelist, the easiest cheat is to bypass
{{notelist}}
entirely and call
{{reflist}}
directly with the group parameter:
{{reflist|group=lower-alpha}}
That often renders correctly even when
{{notelist}}
is broken, because it skips the extra wrapper logic.
If that still shows "lower-alpha" literally, the next cheat is renaming the group to something with built-in MediaWiki styling support. MediaWiki natively recognizes these group names and auto-styles them without any template:
<ref group="lower-alpha">your note</ref>
collected by:
<references group="lower-alpha"/>
The key distinction — using bare
<references/>
instead of a template — lets MediaWiki's own rendering handle the alphabetic styling natively, cutting the templates out entirely.
That second approach is usually the cleanest fix on imported wikis where the template chain is incomplete, since it has zero dependencies.

---

I did the imports (deep clones) and it still didn't work

---

If deep clones didn't fix it, the culprit is almost certainly missing
TemplateStyles
.
{{efn}}
/
{{notelist}}
on Wikipedia rely on a
{{reflist/styles.css}}
TemplateStyles page that often doesn't get pulled in by a normal template clone.
Check if your wiki has:
Template:Reflist/styles.css
Template:Efn/styles.css
These are separate CSS pages that live in the Template namespace and need to be imported independently — they won't show up as dependencies in the template transclusion tree so cloners miss them constantly.
Quick test:
look at your rendered notelist — if the notes are
there
but labeled wrong (showing "lower-alpha 1" etc.), it's styling/CSS. If the notes aren't rendering at all, it's still a Lua or template issue.
If TemplateStyles aren't supported
on your wiki version (requires MediaWiki 1.32+), that's a dead end and you'd need to add the CSS manually to
MediaWiki:Common.css
:
css
ol.references li .reference-text { /* etc */ }
.mw-ref-group-lower-alpha > a::before {
  content: counter(ref, lower-alpha);
}
Which version of MediaWiki are you running, and is TemplateStyles extension installed? That'll confirm which path you're on.
