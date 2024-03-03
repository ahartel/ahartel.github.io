+++
title = "Solution to the Bowling TDD Kata"
date = 2024-03-03

[taxonomies]
tags = ["tdd", "code", "software engineering"]
+++

In a [recent blog post](@2024-03-03-tdd-katas.md) I have collected a couple of TDD
Katas. In this post, I would like to post a solution I found to the Bowling Game Kata.

Uncle Bob breaks this kata down into the following five tests:

1. Gutter game scores zero - When you roll all misses, you get a total score of zero.
2. All ones scores 20 - When you knock down one pin with each ball, your total score is 20.
3. A spare in the first frame, followed by three pins, followed by all misses scores 16.
4. A strike in the first frame, followed by three and then four pins, followed by all misses, scores 24.
5. A perfect game (12 strikes) scores 300.

What follows is a Rust implementation that solves these tests in this order and
describes the evolution of the code.

<!-- more -->

```Rust
use itertools::Itertools;

fn score(rolls: Vec<(u32, u32)>) -> u32 {
    vec![(0, 0), (0, 0)]
        .into_iter()
        .chain(rolls.into_iter())
        .tuple_windows()
        .map(|(a, b, c)| {
            if a.0 == 10 && b.0 == 10 {
                a.0 + b.0 + c.0
            }
            else if b.0 == 10 {
                2 * c.0 + 2 * c.1
            }
            else if b.0 + b.1 == 10 {
                2 * c.0 + c.1
            } else {
                c.0 + c.1
            }
        })
        .sum()
}

/// 1. Gutter game scores zero
///    When you roll all misses, you get a total score of zero.
#[test]
fn gutter_game_scores_zero() {
    assert_eq!(score(vec![]), 0);
}

// First test was super easy. Just introduce the function and return 0.

/// 2. All ones scores 20
///    When you knock down one pin with each ball, your total score is 20.
#[test]
fn all_ones_scores_twenty() {
    assert_eq!(score(vec![(1, 1); 10]), 20);
}

// Second test also still easy. Just have the score function return the sum
// of the elements in the vector.

/// 3. A spare in the first frame, followed by three pins,
///    followed by all misses scores 16.
#[test]
fn a_spare_and_three_scores_16() {
    assert_eq!(score(vec![(5, 5), (3, 0), (0, 0)]), 16)
}

// Third test is more interesting. Now we need to keep tracking state.
// Already, the question arises whether we want to introduce a frame
// abstraction. Could also just iterate over the vector in tuples.
// So the first thing I needed to do was to refactor the implementation
// after the second test to iterate over tuples of 3 rolls.
// From there it was pretty easy, I could just count values double
// whose previous to frames add up to 10.

/// 4. A strike in the first frame, followed by three and then four pins,
///    followed by all misses, scores 24.
#[test]
fn a_strike_then_three_then_four_scores_24() {
    assert_eq!(score(vec![(10, 0), (3, 4), (0, 0)]), 24)
}

// For the forth test I feel it makes sense to introduce a frame as its own
// unit and not just iterate over rolls in a flat way. So first step
// is to refactor the previous code into that.
// After this it was just one more if for the strike case.

/// 5. A perfect game (12 strikes) scores 300.
#[test]
fn a_perfect_game_scores_300() {
    assert_eq!(score(vec![(10, 0); 11]), 300)
}

// First need to refactor this into iterating over a tuple window of 3.
// From there on it was rather easy to just cover the special case that
// frames a and b were both strikes.
```