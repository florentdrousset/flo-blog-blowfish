---
title: "Learning Ocaml"
date: 2023-11-12
draft: false
language: "en"
---

# OCaml, my 🐫

I've started learning OCaml very recently and, oh boy, it's been a hell of a ride.
The language I've worked with the most in my life is PHP. I've studied a bit of C and Java at school, and as a web developer I obviously can't avoid coding in Javascript, but most of what I've learned at school and used at work was (and still is) PHP. Which means that the functional paradigm has been absolutely unknown to me until then.

I enjoy programming on my free time, but it's sometimes difficult to stay focused on a goal when there are no deadlines or obligations attached to that goal. I've also noticed that I get bored pretty quick whenever I try to learn a new language that has features/paradigm similar to PHP : it's not so exciting to rewrite the same thing or do the same projects with just a different syntax.
Another thing that feels exciting is that the OCaml ecosystem for web is not as developped as other more mainstream languages. It feels like there are styll many things to do and lots of code to write, which feels great to me.

I came across OCaml code for the first time very recently and was hooked by the expressive syntax of the language. Being used to imperative languages and not having a very long academic background makes learning OCaml quite tricky, to say the least, but it also makes the process very exciting, and it feels like discovering a whole new world.

But enough said about me : the goal of the following serie of articles is to provey a very quick introduction to the language, from the eyes of a web developer used to PHP.

I will spare you the basic details that you can easily find on Wikipedia (or even better : ocaml.org), but OCaml (Objective Caml) is the last implemenation of the language Caml, that was created in France (cocorico !) in 1987.

# Installation

The first thing that you will need is opam, the package manager for OCaml (that's like composer, _right ??_)
```
bash -c "sh <(curl -fsSL https://raw.githubusercontent.com/ocaml/opam/master/shell/install.sh)"
```

You can then initialize opam, which will create an _opan switch_ with :
```
opam init
```

Answer YES when asked if you want a hook to your shell.

An opam switch is an installation of an ocaml which comes with several parameters : compiler version, installed packages, etc. You can easily switch between different "ocaml environments" (my words, that might not be the proper terminology) by doing
```
opam switch <switch-name>
```

If you just type `opam switch`, you'll see all the switches you have available.

You can then install packages like this :

```
opam install dune
```

By running this command, you'll install [https://github.com/ocaml/dune](dune), the build system for OCaml.
Other interesting packages to install would be [https://github.com/ocaml/merlin](merlin) (auto-completion for vim/emacs), and also [https://opam.ocaml.org/blog/about-utop/](utop) (an improved interface for the toplevel REPL for OCaml) 

---

This is just the first article on a serie. Now that a basic environment has been set up, we'll start writing some actual code in the next article. 