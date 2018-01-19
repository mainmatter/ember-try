# ember-try

[![npm version](https://badge.fury.io/js/ember-try.svg)](https://badge.fury.io/js/ember-try) [![Build Status](https://travis-ci.org/ember-cli/ember-try.svg?branch=master)](https://travis-ci.org/ember-cli/ember-try) [![Ember Observer Score](http://emberobserver.com/badges/ember-try.svg)](http://emberobserver.com/addons/ember-try) [![Build status](https://ci.appveyor.com/api/projects/status/9sswkni8pfuvo4dv/branch/master?svg=true)](https://ci.appveyor.com/project/kategengler/ember-try/branch/master) [![Code Climate](https://codeclimate.com/github/ember-cli/ember-try/badges/gpa.svg)](https://codeclimate.com/github/ember-cli/ember-try) [![Test Coverage](https://codecov.io/gh/ember-cli/ember-try/branch/master/graph/badge.svg)](https://codecov.io/gh/ember-cli/ember-try)

An ember-cli addon to test against multiple bower and npm dependencies, such as `ember` and `ember-data`.

### Installation

```
ember install ember-try
```

### Usage

This addon provides a few commands:

### `ember try:each`

This command will run `ember test` or the configured command with each scenario's specified in the config and exit appropriately.

This command is especially useful to use on CI to test against multiple `ember` versions.

In order to use an alternate config path or to group various scenarios together in a single `try:each` run, you can use
the `--config-path` option.

```
  ember try:each --config-path="config/legacy-scenarios.js"
```

If you need to know the scenario that is being run (i.e. to customize a test output file name) you can use the `EMBER_TRY_CURRENT_SCENARIO`
environment variable.

#### `ember try:one <scenario> (...options) --- <command (Default: ember test)>`

This command will run any `ember-cli` command with the specified scenario. The command will default to `ember test`, if no command is specified on the command-line or in configuration.

For example:

```
  ember try:one ember-1.11-with-ember-data-beta-16 --- ember test --reporter xunit
```

or

```
  ember try:one ember-1.11-with-ember-data-beta-16 --- ember serve
```

When running in a CI environment where changes are discarded you can skip resetting your environment back to its original state by specifying --skip-cleanup=true as an option to ember try.
*Warning: If you use this option and, without cleaning up, build and deploy as the result of a passing test suite, it will build with the last set of dependencies ember try was run with.*

```
  ember try:one ember-1.11 --skip-cleanup=true --- ember test
```

In order to use an alternate config path or to group various scenarios, you can use the `--config-path` option.

```
  ember try:one ember-1.13 --config-path="config/legacy-scenarios.js"
```

#### `ember try:reset`

This command restores the original `bower.json` from `bower.json.ember-try`, `package.json` from `package.json.ember-try`, `rm -rf`s `bower_components` and `node_components` and runs `bower install` and `npm install`. For use if any of the other commands fail to clean up after (they run this by default on completion).

#### `ember try:ember <semver-string>`

Runs `ember test` or the command in config for each version of Ember that is possible under the semver string given. Configuration follows the rules given under the `versionCompatibility` heading below.

#### `ember try:config`

Displays the configuration that will be used. Also takes an optional `--config-path`.

#### (DEPRECATED) `ember try:testall`

This command was renamed to `ember try:each` to better reflect what it does. This command still works, though.

#### (DEPRECATED) `ember try <scenario> <command (Default: test)>`

This command is deprecated in favor of `ember try:one`. There are several bugs with passing options to the specified command that will not be fixed.


### Config

##### versionCompatibility
If you're using `ember-try` with an Ember addon, there is a short cut to test many Ember versions. In your `package.json` under the `ember-addon` key, add the following:

```json
  "ember-addon": {
    "versionCompatibility": {
       "ember": ">1.11.0 <=2.0.0"
    }
  }
```

The value for "ember" can be any valid [semver statement](https://github.com/npm/node-semver).
This will autogenerate scenarios for each version of Ember that matches the statement. It will also include scenarios for `beta` and `canary` channels of Ember that will be allowed to fail.
These scenarios will ONLY be used if `scenarios` is NOT a key in the configuration file being used.
If `useVersionCompatibility` is set to `true` in the config file, the autogenerated scenarios will deep merge with any scenarios in the config file. For example, you could override just the `allowedToFail` property of the `ember-beta` scenario.

To keep this from getting out of hand, `ember-try` will limit the versions of Ember used to the lasted point release per minor version. For example, ">1.11.0 <=2.0.0", would (as of writing) run with versions ['1.11.4', '1.12.2', '1.13.13', '2.0.0'].

##### Configuration Files

Configuration will be read from a file in your ember app in `config/ember-try.js`. Here are the possible options:

```js
/*jshint node:true*/

module.exports = function() {
  return {
    /*
      `command` - a single command that, if set, will be the default command used by `ember-try`.
      P.S. The command doesn't need to be an `ember <something>` command, they can be anything.
      Keep in mind that this config file is JavaScript, so you can code in here to determine the command.
    */
    command: 'ember test --reporter xunit',
    /*
      `bowerOptions` - options to be passed to `bower`.
    */
    bowerOptions: ['--allow-root=true'],
    /*
      `npmOptions` - options to be passed to `npm`.
    */
    npmOptions: ['--loglevel=silent', '--no-shrinkwrap=true'],
    /*
      If set to true, the `versionCompatibility` key under `ember-addon` in `package.json` will be used to
      automatically generate scenarios that will deep merge with any in this configuration file.
    */
    useVersionCompatibility: true,
    scenarios: [
      {
        name: 'Ember 1.10 with ember-data',

        /*
          `command` can also be overridden at the scenario level.
        */
        command: 'ember test --filter ember-1-10',
        bower: {
          dependencies: {
            'ember': '1.10.0',
            'ember-data': '1.0.0-beta.15'
          }
        },
      },
      {
        name: 'Ember 1.11.0-beta.5',
        bower: {
          dependencies: {
            'ember': '1.11.0-beta.5'
          }
        }
      },
      {
        name: 'Ember canary with Ember-Data 2.3.0',
        /*
          `allowedToFail` - If true, if this scenario fails it will not fail the entire try command.
        */
        allowedToFail: true,
        npm: {
          devDependencies: {
            'ember-data': '2.3.0',

            // you can remove any package by marking `null`
            'some-optional-package': null
          }
        },
        bower: {
          dependencies: {
            'ember': 'components/ember#canary'
          },
          resolutions: {
            'ember': 'canary'
          }
        }
      },
      {
        name: 'Ember beta',
        bower: {
          dependencies: {
            'ember': 'components/ember#beta'
          },
          resolutions: { // Resolutions are only necessary when they do not match the version specified in `dependencies`
            'ember': 'beta'
          }
        }
      }
    ]
  };
};
```

Scenarios are sets of dependencies (`bower` and `npm` only). They can be specified exactly as in the `bower.json` or `package.json`
The `name` can be used to try just one scenario using the `ember try:one` command.

If no `config/ember-try.js` file is present, the default config will be used. This is the current default config:

```js
{
  scenarios: [
    {
      name: 'default',
      bower: {
        dependencies: { } /* No dependencies needed as the
                             default is already specified in
                             the consuming app's bower.json */
      }
    },
    {
      name: 'ember-release',
      bower: {
        dependencies: {
          ember: 'release'
        }
      }
    },
    {
      name: 'ember-beta',
      bower: {
        dependencies: {
          ember: 'beta'
        }
      }
    },
    {
      name: 'ember-canary',
      bower: {
        dependencies: {
          ember: 'canary'
        }
      }
    }
  ]
}
```

##### Yarn

If you include `useYarn: true` in your `ember-try` config, all npm scenarios will use `yarn` for install with the `--no-lockfile` option. At cleanup, your dependencies will be restored to their prior state.
 
##### A note on npm scenarios with lockfiles

Lockfiles are ignored by `ember-try`. (`yarn` will run with `--no-lockfile` and `npm` will be run with `--no-shrinkwrap`).
When testing various scenarios, it's important to "float" dependencies so that the scenarios are run with the latest satisfying versions of dependencies a user of the project would get.

### Video
[![How to use EmberTry](https://i.vimeocdn.com/video/559399937_500.jpg)](https://vimeo.com/157688157)

See an example of using `ember-try` for CI [here](https://github.com/kategengler/ember-feature-flags/commit/aaf0226975c76630c875cf6b923fdc23b025aa79), and the resulting build [output](https://travis-ci.org/kategengler/ember-feature-flags/builds/55597086).

### Special Thanks

- Much credit is due to [Edward Faulkner](https://github.com/ef4) The scripts in [liquid-fire](https://github.com/ef4/liquid-fire) that test against multiple ember versions were the inspiration for this project.


### Developing

- Be sure to run `npm link` and `npm link ember-try`, otherwise any `ember try` commands you run will use the version of ember-try included by ember-cli itself.
