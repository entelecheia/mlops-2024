# Home

[![halla-img]][halla-url]
[![course-img]][course-url]
[![lecture-img]][lecture-url]
[![pypi-image]][pypi-url]
[![release-date-image]][release-url]
[![license-image]][license-url]
[![codecov][codecov-image]][codecov-url]

<!-- Links: -->

[halla-img]: https://img.shields.io/badge/CHU-halla.ai-blue
[halla-url]: https://halla.ai
[course-img]: https://img.shields.io/badge/course-entelecheia.ai-blue
[course-url]: https://course.entelecheia.ai
[lecture-img]: https://img.shields.io/badge/lecture-entelecheia.ai-blue
[lecture-url]: https://lecture.entelecheia.ai
[codecov-image]: https://codecov.io/gh/chu-aie/mlops-2024/branch/main/graph/badge.svg?token=6OxfwdlW4Y
[codecov-url]: https://codecov.io/gh/chu-aie/mlops-2024
[pypi-image]: https://img.shields.io/pypi/v/mlops2024
[license-image]: https://img.shields.io/github/license/chu-aie/mlops-2024
[license-url]: https://github.com/chu-aie/mlops-2024/blob/main/LICENSE
[version-image]: https://img.shields.io/github/v/release/chu-aie/mlops-2024?sort=semver
[release-date-image]: https://img.shields.io/github/release-date/chu-aie/mlops-2024
[release-url]: https://github.com/chu-aie/mlops-2024/releases
[jupyter-book-image]: https://jupyterbook.org/en/stable/_images/badge.svg
[repo-url]: https://github.com/chu-aie/mlops-2024
[pypi-url]: https://pypi.org/project/mlops2024
[docs-url]: https://mlops2024.halla.ai
[changelog]: https://github.com/chu-aie/mlops-2024/blob/main/CHANGELOG.md
[contributing guidelines]: https://github.com/chu-aie/mlops-2024/blob/main/CONTRIBUTING.md

<!-- Links: -->

이 수업은 실제 운영 환경에서 기계 학습 시스템의 설계, 구현 및 관리에 대한 실질적인 측면을 다룹니다. 본 과정은 학생들에게 기계 학습 워크플로를 기존 소프트웨어 개발 및 운영 프로세스와 원활하게 통합하는 데 필요한 기술을 제공합니다. 이 과정에서는 DevOps, MLOps 및 보안 관행을 강조합니다. 학생들은 실습 프로젝트를 통해 dot 파일, git, Docker, Kubernetes, CI/CD 파이프라인, Weights & Biases 등 첨단 도구와 방법론을 사용하여 강력한 기계 학습 모델을 구축, 배포하고 유지 관리합니다. 또한, 이 과정은 학생들에게 GitOps, DevSecOps 및 LLMOps의 새로운 분야를 소개하여 실제 기계 학습 애플리케이션의 복잡성에 대비합니다.

```{tableofcontents}

```

## Installation

To install the MLOps 2024 package, use the following command:

```
pip install mlops2024
```

Or

```
pip install --user mlops2024
```

The `--user` flag is optional and can be used to install the package in the user's home directory instead of the system-wide location.

## Usage

To use the MLOps 2024 CLI, run the following command:

```
mlops2024 [OPTIONS]
```

If no option is provided, the website of the book will open in the default web browser.

### Options

The following options are available:

- `--version`: Show the version of the package and exit.
- `-b`, `--build`: Build the book.
- `-l`, `--local`: Open the locally built HTML version of the book in the browser.
- `--help`: Show the help message and exit.

### Examples

1. To build the book, use the following command:

   ```
   mlops2024 --build
   ```

   This will trigger the build process for the book.

2. To open the locally built HTML version of the book in the browser, use the following command:

   ```
   mlops2024 --local
   ```

   This will open the book's HTML file in your default web browser.

3. To view the version of the package, use the following command:

   ```
   mlops2024 --version
   ```

   This will display the version number of the MLOps 2024 package.

For more information and additional options, run `mlops2024 --help` to see the help message.

## Contributors

<!-- ALL-CONTRIBUTORS-LIST:START - Do not remove or modify this section -->
<!-- prettier-ignore-start -->
<!-- markdownlint-disable -->

<!-- markdownlint-restore -->
<!-- prettier-ignore-end -->

<!-- ALL-CONTRIBUTORS-LIST:END -->

## Changelog

See the [CHANGELOG] for more information.

## Contributing

Contributions are welcome! Please see the [contributing guidelines] for more information.

## License

This project is released under the [CC-BY-4.0 License][license-url].
