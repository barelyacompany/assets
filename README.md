# barelyacompany/assets

Central release assets repository for all company-owned products. Source repos check this repo out directly during their release workflow and push versioned config files here.

## Structure

```
{repo-name}/
  {version}/
    *.yaml    # config files copied from the source repo at release time
```

Example after a locko-operator release:
```
locko-operator/
  v1.0.0/
    lockoconfig.yaml
    role.yaml
    role_binding.yaml
    operator.yaml
```

## How source repos publish here

Source repos use `RELEASES_PAT` (already available as an org/repo secret) to check out this repo, copy their release artifacts into a versioned folder, and push.

### Standard release job pattern

```yaml
publish-assets:
  runs-on: ubuntu-latest
  needs: build-and-push
  permissions:
    contents: read

  steps:
    - name: Checkout source
      uses: actions/checkout@v4

    - name: Resolve version
      id: version
      run: |
        RAW="${{ github.event_name == 'workflow_dispatch' && inputs.tag || github.ref_name }}"
        echo "tag=v${RAW#v}" >> $GITHUB_OUTPUT

    - name: Checkout assets repo
      uses: actions/checkout@v4
      with:
        repository: barelyacompany/assets
        token: ${{ secrets.RELEASES_PAT }}
        path: assets-repo

    - name: Copy files into versioned folder
      run: |
        VERSION=${{ steps.version.outputs.tag }}
        DEST=assets-repo/{repo-name}/$VERSION
        mkdir -p $DEST
        cp config/path/to/file.yaml $DEST/
        # add more files as needed

    - name: Commit and push
      run: |
        VERSION=${{ steps.version.outputs.tag }}
        cd assets-repo
        git config user.name "github-actions[bot]"
        git config user.email "github-actions[bot]@users.noreply.github.com"
        git add {repo-name}/$VERSION/
        git commit -m "{repo-name} $VERSION"
        git push
```

## Manual sync

If a release needs to be backfilled, use the **Manual Release Sync** workflow from the Actions tab. It checks out the source repo at the given tag and copies config files automatically.
