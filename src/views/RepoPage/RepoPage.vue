<style lang="scss">
@import '../../styles/carbon-utils';
.repo-page .bx--row {
  padding-top: $spacing-05;
  padding-bottom: $spacing-05;
}
</style>

<script>
// eslint-disable-next-line no-unused-vars
import RepoTable from './RepoTable';
import gql from 'graphql-tag';
const REPO_QUERY = gql`
  query REPO_QUERY {
    # Let's use carbon as our organization
    organization(login: "carbon-design-system") {
      # We'll grab all the repositories in one go. To load more resources
      # continuously, see the advanced topics.
      repositories(first: 75, orderBy: { field: UPDATED_AT, direction: DESC }) {
        totalCount
        nodes {
          url
          homepageUrl
          issues(filterBy: { states: OPEN }) {
            totalCount
          }
          stargazers {
            totalCount
          }
          releases(first: 1) {
            totalCount
            nodes {
              name
            }
          }
          name
          updatedAt
          createdAt
          description
          id
        }
      }
    }
  }
`;
const headers = [
  { key: 'name', header: 'Name' },
  { key: 'createdAt', header: 'Created' },
  { key: 'updatedAt', header: 'Updated' },
  { key: 'issueCount', header: 'Open Issues' },
  { key: 'stars', header: 'Stars' },
  { key: 'links', header: 'Links' }
];
// const rows = [
//   {
//     id: '1',
//     name: 'Repo 1',
//     createdAt: 'Date',
//     updatedAt: 'Date',
//     issueCount: '123',
//     stars: '456',
//     links: 'Links'
//   },
//   {
//     id: '2',
//     name: 'Repo 2',
//     createdAt: 'Date',
//     updatedAt: 'Date',
//     issueCount: '123',
//     stars: '456',
//     links: 'Links'
//   },
//   {
//     id: '3',
//     name: 'Repo 3',
//     createdAt: 'Date',
//     updatedAt: 'Date',
//     issueCount: '123',
//     stars: '456',
//     links: 'Links'
//   }
// ];
export default {
  name: 'RepoPage',
  components: { RepoTable },
  data() {
    return {
      headers,
      pageSize: 0,
      pageStart: 0,
      page: 0
    };
  },
  computed: {
    rows() {
      if (!this.organization) {
        return [];
      } else {
        return this.organization.repositories.nodes.map(row => ({
          ...row,
          key: row.id,
          stars: row.stargazers.totalCount,
          issueCount: row.issues.totalCount,
          createdAt: new Date(row.createdAt).toLocaleDateString(),
          updatedAt: new Date(row.updatedAt).toLocaleDateString(),
          links: { url: row.url, homepageUrl: row.homepageUrl }
        }));
      }
    },
    pagedRows() {
      return this.rows.slice(this.pageStart, this.pageStart + this.pageSize);
    }
  },
  methods: {
    onPagination(val) {
      this.pageSize = val.length;
      this.pageStart = val.start;
      this.page = val.page;
    }
  },
  apollo: {
    organization: REPO_QUERY
  }
};
</script>

<template>
  <div class="bx--grid bx--grid--full-width bx--grid--no-gutter repo-page">
    <div class="bx--row repo-page__r1">
      <div class="bx--col-lg-16" style="width: 100%">
        <!-- {{ this.organization }} -->
        <repo-table
          :headers="headers"
          :rows="pagedRows"
          :totalRows="rows.length"
          @pagination="onPagination"
          title="Carbon Repositories"
          helperText="A collection of public Carbon repositories."
          :loading="$apollo.loading"
        />
      </div>
    </div>
  </div>
</template>
