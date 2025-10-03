First, a package is initialized with help of {usethis}:

```{R REPL}
# Package names cannot have _
# Using . in the name may be confusing as well due to class methods
usethis::create_package("~/path/to/package/inethemes")
```

Setting up a README and Git repo:

```{R REPL}
usethis::use_readme_md()
```

```{R REPL}
usethis::use_git()
```

`usethis::use_r()` to create an R file under `R/`:

```{R REPL}
usethis::use_r("ine_colors")
```

We define our color palette and `@export` it to end users of our package:

```{R R/ine_colors.R}
#' @export
ine_colors <- list(
	cold = c("#cce6ff", "#66afff", "#2f3d61", "#181a2f", "#a08b65")
	warm = c("#fff0cc", "#ffbb66", "#ff8c33", "#76431b", "#382712")
)
```

Now, we create a constructor for the palette:

```{R REPL}
usethis::use_r("ine_palette")
```

```{R R/ine_palete.R}
#' Palette constructor function
#'
#' @param option Name of palette
#' @param n Number of colors
#'
#' @returns A character vector of hex codes
#' @export
ine_palette <- function(option = "blue", n = NULL) {
	stopifnot(is.null(option) || option %in% ine_colors |> names())

	if (is.null(n)) {
		return(ine_colors[[option]])
	}

	if (n > length(ine_colors[[option]])) {
		stop(paste0("Not enough colors in palette ", option))
	}

	ine_colors[[option]][1:n]
}
```

Now, the {ggplot2} extension, always noted with `ggplot2::` notation for namespace conflicts avoidance and better dependency management:

```{R REPL}
usethis::use_r("scale_fill_ine")
```

```{R R/scale_fill_ine.R}
#' Scale fill wrapper
#'
#' @param option Name of palette
#' @param ... remaining custom args
#'
#' @returns a ggplot2 function
#' @export
scale_fill_ine <- function(option = "blue", ...) {
	ggplot2::discrete_scale(
		aesthetics = "fill",
		scale_name = option,
		palette = function(n) ine_palette(option, n),
		...
	)
}
```

To test our code, first we document our package for R to write doc files and know which objects to export to the end user, load our package onto memory (equivalent to library(our_package)), then test it on real data.

NOTE: Writing tests with `usethis::use_testthat()` followed by `usethis::use_test("some_of_our_defined_functions")` is encouraged, but not done for purposes of presentation.

```{R REPL}
devtools::document()
```

```{R REPL}
devtools::load_all()
```

```{R example.R}
library(ggplot2)

# Example dataset
df <- data.frame(
	group = rep(LETTERS[1:5], each = 3),
	x = rep(1:3, times = 5),
	y = runif(15, 1, 10)
)

df |>
	ggplot(aes(x, y, fill = group)) +geom_col(position = "dodge") +
	inethemes::scale_fill_ine(option = "cold") +
	theme_minimal()
```
