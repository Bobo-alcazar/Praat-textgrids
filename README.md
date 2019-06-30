# praat-textgrids -- Praat TextGrid manipulation in Python

## Description

`textgrids` is a module for handling Praat short or long text-file TextGrid files. It implements five classes. From largest to smallest:

* `TextGrid` -- a `dict` with tier names as keys and `Tier`s as values
* `Tier` -- a `list` of either `Interval` or `Point` objects
* `Interval` -- an `object` representing Praat intervals
* `Point` -- a `namedtuple` representing Praat points
* `Transcript` -- a `str` with special methods for transcription handling

All Praat text objects are represented as `Transcript` objects.

## Copyright

Copyright © 2019 Legisign.org, Tommi Nieminen <software@legisign.org>

This program is free software: you can redistribute it and/or modify it under the terms of the GNU General Public License as published by the Free Software Foundation, either version 3 of the License, or (at your option) any later version.

This program is distributed in the hope that it will be useful, but WITHOUT ANY WARRANTY; without even the implied warranty of MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE. See the GNU General Public License for more details.

You should have received a copy of the GNU General Public License along with this program.  If not, see <https://www.gnu.org/licenses/>.

## Module contents

### 1. TextGrid

`TextGrid` is a `dict` whose keys are tier names (strings) and values are `Tier` objects. The constructor takes an optional filename argument for easy loading and parsing textgrid files.

#### 1.1. Properties

All the properties of `dict`s plus:

* `filename` holds the textgrid filename, if any. `read()` and `write()` methods both set or update it.

#### 1.2. Methods

All the methods of `dict`s plus:

* `parse()` -- parse string `data` into a TextGrid
* `read()` -- read a TextGrid file `name`
* `write()` -- write a TextGrid file `name`
* `tier_from_csv()` -- read a textgrid tier from a CSV file
* `tier_to_csv()` -- write a textgrid tier into a CSV file

`parse()`, `read()` and `write()` each take a filename argument.

`tier_from_csv()` and `tier_to_csv()` both take two obligatory arguments, the tier name and the filename, in that order. A Boolean `point_tier=True` can be given to `tier_from_csv()` to read point tiers.

### 2. Tier

`Tier` is a list of either `Interval` or `Point` objects.

**NOTE:** `Tier` only allows adding `Interval` or `Point` objects. Adding anything else or mixing `Interval`s and `Point`s will trigger an exception.

#### 2.2. Properties

All the properties of `list`s plus:

* `is_point_tier` -- Boolean value: `True` for point tier, `False` for interval tier.

#### 2.3. Methods

All the methods of `list`s plus:

* `concat()` -- concatenate intervals
* `to_csv()` -- convert tier data into a CSV-like list

`concat()` returns a `TypeError` if used with a point tier. It takes two optional arguments, `first=` and `last=`, both of which are integer indexes with the usual Python semantics: 0 stands for the first element, -1 for the last element, these being also the defaults.

`to_csv()` returns a CSV-like list. It’s mainly intended to be used from the `TextGrid` level method `tier_to_csv()` but can be called directly if writing to a file is not desired.

### 3. Interval

`Interval` is an `object` class.

#### 3.1. Properties

* `dur` -- interval duration (`float`)
* `mid` -- interval midpoint (`float`)
* `text` -- text label (`Transcript`)
* `xmax` -- interval end time (`float`)
* `xmin` -- interval start time (`float`)

#### 3.3. Methods

* `containsvowel()` -- Boolean: does the interval contain a vowel?
* `startswithvowel()` -- Boolean: does the interval start with a vowel?
* `timegrid()` -- create a grid of even time slices

`containsvowel()` and `startswithvowel()` check for possible vowels in both Praat notation and Unicode but can of course make an error if symbols are used in an unexpected way. They don’t take arguments.

`timegrid()` returns a list of timepoints (in `float`) evenly distributed from `xmin` to `xmax`. It takes an optional integer argument specifying the number of timepoints desired; the default is 3. It raises a `ValueError` if the argument is not an integer or is less than 1.

### 4. Point

`Point` is a `namedtuple` with two properties: `text` and `xpos`.

#### 4.1. Properties

* `text` -- text label (`Transcript`)
* `xpos` -- temporal position (`float`)

### 5. Transcript

`Transcript` is a `str`-derived class with one special method: `transcode()`.

### 5.1. Properties

All the properties of `str`s.

#### 5.2. Methods

All the methods of `str`s plus:

* `transcode()` -- convert Praat notation to Unicode or vice versa.

Without arguments, `transcode()` assumes its input to be in Praat notation and converts it to Unicode; no check is made as to whether the input really is in Praat notation but nothing will happen if it isn’t.

Optional `to_unicode=False` argument inverts the direction of the transcoding from Unicode to Praat. Again, it is not checked whether input is in Unicode.

With optional `retain_diacritics=True` argument (only applicable with `to_unicode=True` which is the default direction) the transcoding does not remove over- and understrike diacritics from the transcript.

## Example code

    import sys
    import textgrids

    for arg in sys.argv[1:]:
        # Try to open the file as textgrid
        try:
            grid = textgrids.TextGrid(arg)
        # Discard and try the next one
        except:
            continue

        # Assume "syllables" is the name of the tier
        # containing syllable information
        for syll in grid['syllables']:
            # Convert Praat to Unicode in the label
            label = syll.text.transcode()
            # Print label and syllable duration, CSV-like
            print('"{}";{}'.format(label, syll.dur))
