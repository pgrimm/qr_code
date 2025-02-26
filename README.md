# QRCode

[![Continuous Integration](https://github.com/iodevs/qr_code/workflows/Continuous%20Integration/badge.svg)](https://github.com/iodevs/qr_code/actions)
[![Coverage Status](https://coveralls.io/repos/github/iodevs/qr_code/badge.svg?branch=master)](https://coveralls.io/github/iodevs/qr_code?branch=master)
![GitHub top language](https://img.shields.io/github/languages/top/iodevs/qr_code)
![Hex.pm](https://img.shields.io/hexpm/v/qr_code)
![Hex.pm](https://img.shields.io/hexpm/dt/qr_code)

This library is useful for generating QR code to your projects.

![QR code](docs/qrcode.svg)

## Installation

```elixir
def deps do
  [
    {:qr_code, "~> 2.2.1"}
  ]
end
```

## Usage

If you want to create QR code, just use the function `QRCode.create(your_string, level)`:

```elixir
  iex> QRCode.create("Hello World")
  {:ok, %QRCode.QR{...}}
```

You can also change the error correction level according to your needs. There are four level of error corrections:

```elixir
  | Error Correction Level    | Error Correction Capability    |
  |---------------------------|--------------------------------|
  | :low (default value)      | recovers 7% of data            |
  | :medium                   | recovers 15% of data           |
  | :quartile                 | recovers 25% of data           |
  | :high                     | recovers 30% of data           |
```

> Be aware higher levels of error correction require more bytes, so the higher the error correction level,
> the larger the QR code will have to be.

We've just generated QR code and now we want to save it to some image file format with high quality. We can do
that by using `QRCode.Svg.save_as/3` function:

```elixir
  iex> "Hello World"
        |> QRCode.create(:high)
        |> Result.and_then(&QRCode.Svg.save_as(&1,"/path/to/hello.svg"))
  {:ok, "/path/to/hello.svg"}
```

where we used an error correction level `:high` and our library [Result](https://hexdocs.pm/result/api-reference.html).
As you can see the svg file will be saved into `/path/to/` directory.

Also there are a few settings for svg:

```elixir
| Setting            | Type                   | Default value | Description               |
|--------------------|------------------------|---------------|---------------------------|
| scale              | positive integer       | 10            | scale for svg QR code     |
| background_opacity | nil or 0.0 <= x <= 1.0 | nil           | background opacity of svg |
| background_color   | string or {r, g, b}    | "#ffffff"     | background color of svg   |
| qrcode_color       | string or {r, g, b}    | "#000000"     | color of QR code          |
| format             | :none or :indent       | :none         | indentation of elements   |
```

By this option, you can set the size QR code, background color (and also opacity) of QR code or QR code colors. The format option is for removing indentation (of elements like is `<rect.. />`) in a svg file. It means that for value `:none`, the svg file contains only one "line of code" (no indentation), whereas for `:indent` svg file has a structure and svg code is more readable.

Let's see an example below:

```elixir
  iex> settings = %QRCode.SvgSettings{qrcode_color: {17, 170, 136}}
  iex> "your_string"
        |> QRCode.create()
        |> Result.and_then(&QRCode.Svg.save_as(&1,"/tmp/your_name.svg", settings))
  {:ok, "/tmp/your_name.svg"}
```

![QR code color](docs/qrcode_color.svg)

## Limitations

The QR code is limited by characters that can contain. In our case this library was developed only for `Byte` mode.
For example, the limits for **40 version** are:

```elixir
|           |      Maximum number of characters      |
| Level     |   low   |  medium  | quartile |  high  |
|-----------|---------|----------|----------|--------|
| Byte mode |   2953  |   2331   |   1663   |  1273  |
```

For other versions and modes see [Character Capacities](https://www.thonky.com/qr-code-tutorial/character-capacities) in documentation.

If you get an `{:error, "Input string can't be encoded"}` it means that you overcame the limit for the given `version` and `level` (i.e. your string is too big).

If anyone needs the rest of encoding modes,
please open new issue or push your code in this repository.

## Notes

- If you need a png format instead of svg, you can use [mogrify](https://github.com/route/mogrify) to convert it:

  ```elixir
  import Mogrify

  "qr_code.svg"
    |> Mogrify.open()
    |> format("png")
    |> save(path: "qr_code.png")
  ```

- You can also save the QR matrix to csv using by [csvlixir](https://github.com/jimm/csvlixir):

  ```elixir
  {:ok, qr} = QRCode.create("Hello World")
  save_csv(qr.matrix, "qr_matrix.csv")

  def save_csv(matrix, name_file) do
    name_file
    |> File.open([:write], fn file ->
      matrix
      |> CSVLixir.write()
      |> Enum.each(&IO.write(file, &1))
    end)
  end
  ```

## References

- [http://www.thonky.com/qr-code-tutorial/](http://www.thonky.com/qr-code-tutorial/)

## License

QRCode source code is licensed under the _BSD-4-Clause._

---

Created: 2018-11-24Z
