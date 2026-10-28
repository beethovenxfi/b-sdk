# Publishing @beets/b-sdk

The package is configured for public publication on npmjs.com. Its source repository is `beethovenxfi/b-sdk`. The existing version sequence is retained; Changesets manages subsequent version bumps. Historical entries in `CHANGELOG.md` describe upstream releases.

## npm access and first publication

1. Use an npm account with permission to publish packages in the `@beets` scope. Confirm that `@beets/b-sdk` is available or that the account has write access to the existing package.
2. If the package does not exist yet, publish its first version from a reviewed checkout of `main`. Authenticate interactively with npm and satisfy its 2FA requirements:

   ```sh
   pnpm install --frozen-lockfile
   pnpm build
   npm login --registry=https://registry.npmjs.org/
   npm publish --access public --registry=https://registry.npmjs.org/
   ```

   This command publishes for real. Do not run it for validation; use `npm pack --dry-run` to inspect the package without uploading it. Do not republish a version that already exists.

3. In the npm package settings for `@beets/b-sdk`, configure a GitHub Actions trusted publisher:

   - Organization or user: `beethovenxfi`
   - Repository: `b-sdk`
   - Workflow filename: `main.yml` (the caller, not `release.yml`)
   - Environment: leave unset; this workflow does not use a GitHub environment
   - Allow direct publishing with `npm publish`

4. Enable GitHub Actions in the repository and allow Actions to create pull requests in the repository Actions settings. The workflow uses `GITHUB_TOKEN` for release PRs and tags; it does not require an `NPM_TOKEN` secret.

Trusted publishing uses npm OIDC on GitHub-hosted runners. The release workflow runs Node 24 and pins npm 11.5.2, which supports OIDC and remains compatible with the publish arguments forwarded by pnpm/Changesets. npm account permissions and trusted-publisher settings are managed on npmjs.com, not by changing the Git remote.

## Subsequent releases

1. Add a changeset with `pnpm changeset` for a releasable change. Select `@beets/b-sdk`, the version bump, and a release summary.
2. Merge the change into `main`. The Main workflow calls the reusable Release workflow; Changesets creates or updates a version PR.
3. Merge the version PR. The next Main run builds and publishes any version absent from npm using `pnpm changeset:release` (`pnpm build && changeset publish`), then creates release tags.

To retry a release, run the **Main** workflow manually on `main`. Release is reusable-only so both automatic and manual runs use the same npm-trusted caller. Publishing is restricted to `beethovenxfi/b-sdk` on `main`.

The build generates GraphQL types and bundles `src/index.ts` into CJS, ESM, and TypeScript declarations. Only `dist/` plus npm-standard metadata files are packaged. The dependency `@balancer-labs/balancer-maths` keeps its upstream name.

## Validation without publishing

```sh
pnpm check-types
pnpm exec vitest run src/entities/tokenAmount.test.ts
pnpm build
npm pack --dry-run
pnpm changeset status
```

Check that the package name is `@beets/b-sdk`, repository metadata points to `beethovenxfi/b-sdk`, and the tarball contains the entry points declared in `package.json`.
